# Writing Plans Examples

Use these examples to calibrate plan depth and reader surfaces. A plan has two audiences:

- **Human reviewer:** approves intent, scope, ordering, tradeoffs, and risk posture without needing codebase knowledge.
- **Agent executor:** starts cold from exact files, task boundaries, verification commands, and revision triggers.

`SKILL.md` is the source of truth for planning rules; update this file when those rules change. Examples are intentionally few: keep an example only when it teaches a distinct planning judgment.

| Example | Why it stays |
|---------|--------------|
| Tiny bug fix | Shows the minimum viable plan: human-readable intent plus one executable note. |
| Medium webhook feature | Shows full human/agent split, vertical slicing, risk-first ordering, exclusions, stop signals, and executable tasks. |
| Large plugin system | Shows contract-first planning, dependency ordering, progressive refinement, and plan revision after early findings. |
| Completion-faking anti-pattern | Shows a plan that looks structured but fails cold-start execution. |
| Horizontal-slicing anti-pattern | Shows why layer-by-layer tasking delays learning and verification. |

---

## Tiny: Bug Fix

**Situation:** An API returns 404 when an expired token is used. It should return 401.

**Assessment**
- **Size:** Tiny. One condition check in one function; no interface change.
- **Nature:** Bug fix. Root cause identified: `auth_middleware.py:47` checks `token.exists()` before `token.is_valid()`, so expired tokens follow the wrong branch.
- **Current state:** `authenticate()` in `auth_middleware.py` handles token errors; no other callers are affected.
- **Done state:** `pytest tests/test_auth.py::test_expired_token_returns_401 -v` passes.

### Human Review Section

> **Intent:** Fix the authentication bug where expired sessions show users a "not found" error instead of prompting them to log in again.
>
> **Summary (Layer 1)**
> - **Goal:** Expired tokens return Unauthorized instead of Not Found.
> - **Why:** Users should know their session expired, not think the endpoint is missing.
> - **Expected outcome:** Expired-token requests return 401.
> - **Impact scope:** Authentication behavior only.
> - **Size:** Tiny.

### Execution Plan

Root cause: `auth_middleware.py:47` checks token existence before validity. Add the validity check there so expired tokens return 401 before the missing-token path can return 404.

Verify: `pytest tests/test_auth.py::test_expired_token_returns_401 -v` -> PASS.

---

## Medium: Webhook Notifications

**Situation:** When a job completes, the system should POST the job result to a user-configured webhook URL.

**Assessment**
- **Size:** Medium. Introduces a new notification concept and crosses job execution, configuration, HTTP behavior, and tests.
- **Nature:** Feature. No existing webhook system; config and retry decisions are needed.
- **Current state:** `job_runner.py` fires an `on_complete` event; config lives in `config.yaml`; no HTTP client is currently used.
- **Done state:** A completed job sends the expected payload to a configured URL, retries on transient failure, and tests pass.

### Human Review Section

> **Intent:** Let users receive real-time job-completion notifications in their own systems without polling.
>
> **Summary (Layer 1)**
> - **Goal:** Send webhook notifications when jobs complete.
> - **Why:** Users need job results delivered to external systems automatically.
> - **Expected outcome:** A completed job POSTs its result to a configured URL and retries transient failures.
> - **Impact scope:** Job completion behavior, configuration, and notification delivery.
> - **Size:** Medium.
>
> **Summary (Layer 2 - Approach)**
> Build a walking skeleton first: send one hardcoded webhook from the job-completion event before adding configuration and retry behavior. This resolves the main uncertainty early: whether the completion event carries enough data for a useful payload.
>
> **Task Overview**
> - **Task 1:** Prove one webhook can fire from job completion before investing in configuration or retries.
> - **Task 2:** Move the URL into configuration after the event path is proven.
> - **Task 3:** Add retry behavior last, once the sender and config path are stable.
>
> **Key Decisions**
> - **Delivery scope:** Single webhook target vs. multiple targets -> choose one target for this iteration to keep behavior reviewable.
> - **Retry scope:** Simple client-side retry vs. queue-backed delivery -> choose client-side retry because webhook delivery is useful but not critical job infrastructure.
>
> **Conflict Priority**
> When completeness conflicts with a working subset, prefer the working subset. A configurable webhook is more important than retry behavior.

### Execution Plan

**Decomposition strategy**
- **Walking skeleton first:** connect job completion to one POST before adding config or retry.
- **Risk-first ordering:** validate the `on_complete` event payload before building dependent behavior.
- **Vertical slicing:** every task adds verifiable behavior.

**Explicit exclusions**
- No webhook signing or authentication in this iteration.
- No multiple webhook targets per job.
- No webhook management UI; URL is config-only.

**Stop signals**
- If `on_complete` does not provide enough data for a useful payload, pause before designing a workaround.
- If retry requires a new external dependency, pause before adding it.

**File map**
- `job_runner.py`: wire notification into completion handling.
- `config.yaml`: add webhook URL configuration.
- `notifications/`: create webhook sender module.
- `tests/notifications/`: add notification tests.

### Task 1: Walking Skeleton

Proves the completion event can produce one webhook payload end-to-end.

Files: `job_runner.py`; new webhook sender module in `notifications/`; tests in `tests/notifications/`.

- [ ] Create a sender with `notify(job_result)` that POSTs to a hardcoded URL.
- [ ] Mock HTTP and assert the POST payload contains the job result fields.
- [ ] Call the sender from the job-completion path.
- [ ] Run `pytest tests/notifications/ -v` -> PASS.
- [ ] Commit point: a completed job triggers one hardcoded webhook.

### Task 2: Configuration

Makes the proven webhook path usable without code changes.

Files: `config.yaml`; webhook sender from Task 1; `job_runner.py`.

- [ ] Add webhook URL configuration.
- [ ] Read the URL from config instead of using a hardcoded value.
- [ ] Skip webhook creation when the URL is absent.
- [ ] Add a no-config test proving no webhook is sent.
- [ ] Run `pytest tests/ -v` -> PASS.
- [ ] Commit point: webhook delivery is config-driven.

### Task 3: Retry Behavior

Adds resilience after the event path and configuration are stable.

Files: webhook sender and its tests.

- [ ] Retry transient 5xx failures with backoff.
- [ ] Do not retry permanent 4xx failures.
- [ ] Add tests for retry, no-retry, and success-after-retry.
- [ ] Run `pytest tests/notifications/ -v` -> PASS.
- [ ] Commit point: retry behavior is complete.

**Final verification:** Run a test job with the webhook URL pointed at a local echo server and confirm the expected payload is received.

---

## Large: Plugin System

**Situation:** Add a plugin system so users can drop custom processors into a directory. Plugins are discovered, validated, sandboxed, and run as part of the data pipeline.

**Assessment**
- **Size:** Large. Introduces a new architecture concept and affects pipeline execution, configuration, CLI behavior, plugin API, validation, and sandboxing.
- **Nature:** Architecture change. The plugin interface, loading model, sandboxing boundary, and CLI/config contract must be decided before broad implementation.
- **Current state:** `pipeline/runner.py` calls hardcoded processors in sequence; `config.yaml` stores configuration; no dynamic loading exists.
- **Done state:** A sample plugin is discovered, validated, sandboxed, enabled through config/CLI, and executed by the pipeline.

### Human Review Section

> **Intent:** Let users extend the data pipeline with custom processors without changing core pipeline code.
>
> **Summary (Layer 1)**
> - **Goal:** Add a user-extensible plugin system for data processors.
> - **Why:** Users need custom processing behavior without forking or editing the pipeline.
> - **Expected outcome:** Users place plugin files in a directory; the system discovers, validates, sandboxes, enables, and runs them.
> - **Impact scope:** Pipeline execution, plugin infrastructure, configuration, and CLI controls.
> - **Size:** Large.
>
> **Summary (Layer 2 - Approach)**
> Define the plugin contract first, prove it with one hardcoded no-op plugin, then resolve sandboxing before detailing discovery and CLI behavior. Discovery and CLI stay goal-level until sandboxing reveals whether plugins can be inspected in-process.
>
> **Task Overview**
> - **Task 1:** Define the plugin contract so all later work has a stable boundary.
> - **Task 2:** Prove the contract works through the pipeline with one hardcoded plugin.
> - **Task 3:** Decide and implement sandboxing before building dynamic discovery on top of it.
> - **Task 4:** Discover and validate plugins using the sandboxing constraints from Task 3.
> - **Task 5:** Add CLI/config controls after discovery and validation behavior is known.
>
> **Key Decisions**
> - **Plugin contract:** Standard `process` and `validate` methods plus metadata -> chosen because pipeline execution and validation both need explicit entry points.
> - **Sandboxing decision point:** Decide after an early spike, not in the abstract -> chosen because subprocess vs. restricted globals changes discovery, validation, and CLI design.
> - **Progressive refinement:** Keep Tasks 4-5 goal-level until sandboxing is known -> chosen to avoid detailed instructions built on an unproven isolation model.
>
> **Conflict Priority**
> When extensibility conflicts with host-process safety, prefer host-process safety. Unsafe plugin execution would compromise the pipeline for every user.

### Execution Plan

**Decomposition strategy**
- **Contract first:** define the interface before building either side.
- **Walking skeleton:** prove one plugin can run through the pipeline before dynamic loading.
- **Risk first:** sandboxing is the biggest unknown and must be settled before discovery and CLI details.
- **Progressive refinement:** Tasks 1-3 are detailed; Tasks 4-5 stay goal-level until Task 3 completes.

**Risk analysis**
1. Wrong plugin contract forces rework across pipeline, discovery, validation, and CLI. Resolve in Task 1.
2. Sandboxing constraints can change how plugins are loaded and inspected. Resolve in Task 3.
3. Validation may need to happen without importing plugin code in-process. Detail Task 4 only after Task 3.

**Dependency graph**

```text
Task 1: Plugin interface
  -> Task 2: Pipeline walking skeleton
  -> Task 3: Sandboxing
    -> Task 4: Discovery and validation
    -> Task 5: CLI and config controls
```

Tasks 4 and 5 may run in parallel only after Task 3 defines the sandboxing contract and their file ownership is separated.

### Task 1: Plugin Interface Contract

Defines the boundary before building either side of it.

Files: new interface module and `NoOpPlugin` in `plugins/`; tests in `tests/plugins/`.

- [ ] Define plugin interface with `process(data) -> data` and `validate() -> bool`.
- [ ] Define plugin metadata fields: name, version, author, description.
- [ ] Add a no-op plugin that returns data unchanged for testing.
- [ ] Test interface conformance and metadata access.
- [ ] Run `pytest tests/plugins/ -v` -> PASS.
- [ ] Commit point: plugin contract is defined and tested.

### Task 2: Pipeline Walking Skeleton

Proves the contract works through real pipeline execution before dynamic loading exists.

Files: `pipeline/runner.py`; integration tests in `tests/plugins/`.

- [ ] Allow the pipeline runner to accept plugin instances.
- [ ] Wire the no-op plugin from Task 1 into the pipeline.
- [ ] Test that pipeline data passes through the plugin correctly.
- [ ] Run `pytest tests/ -v` -> PASS.
- [ ] Commit point: one hardcoded plugin runs in the pipeline.

### Task 3: Sandboxing Spike and Implementation

Resolves the highest-risk architectural unknown before discovery and CLI are designed in detail.

Files: sandboxing module and tests in `plugins/`.

- [ ] Compare subprocess isolation and restricted globals against filesystem access, timeout, and host-process safety.
- [ ] Implement the chosen sandboxed runner.
- [ ] Test that filesystem access is blocked, timeout is enforced, and plugin crashes do not kill the host process.
- [ ] Run `pytest tests/plugins/ -v` -> PASS.
- [ ] Commit point: sandboxing works and its constraints are documented in the execution log.
- [ ] Revision trigger: after this task, refine Tasks 4-5 based on the sandboxing result.

### Task 4: Discovery and Validation (Goal-Level)

Goal: Load plugin candidates from a configured directory, validate them against the interface, and register only valid plugins.

Depends on: Task 3 sandboxing constraints.

Files: discovery/validation modules in `plugins/`; `config.yaml`. Exact module split depends on Task 3.

Success signal: `pytest tests/plugins/test_discovery.py -v` -> PASS, with valid plugins registered and invalid plugins rejected.

### Task 5: CLI and Config Controls (Goal-Level)

Goal: Let users list, enable, and disable discovered plugins.

Depends on: Task 3 sandboxing constraints and the registration shape from Task 4.

Files: `cli/plugins.py`; `config.yaml`; CLI tests.

Success signal: `pytest tests/cli/test_plugins.py -v` -> PASS, and enabled plugins are the only ones run by the pipeline.

**Final verification:** Place a sample plugin in the plugin directory, run the pipeline, and confirm the plugin is discovered, validated, sandboxed, enabled, and executed.

### Execution Log Example

```text
[2026-04-06] Task 1: complete. `PluginBase`, `PluginMetadata`, and `NoOpPlugin`
were defined. `PipelineData` moved to `plugins/interface.py` to avoid a circular
import: `plugins/` must not import from `pipeline/` at module load time.

[2026-04-07] Task 3: failed first attempt. Restricted globals blocked `open`,
`socket`, and `__import__`, but `pytest tests/plugins/test_sandbox.py::test_filesystem_access`
failed because `importlib.import_module` bypassed the block. Switched to subprocess
isolation with JSON stdin/stdout. `pytest tests/plugins/test_sandbox.py -v` -> PASS.

Plan revision triggered. Because plugins are now out-of-process and stateless from
the host perspective, Task 4 must validate via subprocess and discovery must read
metadata from a manifest file without importing plugin code. Task 5 must operate
through config only; in-process plugin introspection is not available.

[2026-04-08] Task 4: failed. `pytest tests/plugins/test_discovery.py::test_invalid_plugin_rejected`
hung because a plugin's `validate()` called `input()`. Added a 5s sandbox timeout;
timed-out plugins are rejected as invalid. `pytest tests/plugins/ -v` -> PASS.
```

---

## Anti-Pattern: Completion-Faking

A plan can look orderly while failing as an execution artifact.

```markdown
### Task 1: Set up the notifier

This is to set up the notifier component.

Files: `notifications/webhook.py`

- [ ] Create the notifier
- [ ] Write tests
- [ ] Run the tests

### Task 2: Configure it

This is to configure the webhook.

Files: `config.yaml`, `notifications/webhook.py`

- [ ] Add config support
- [ ] Test config behavior
```

**Why this fails**
- The purpose sentences restate the task names instead of explaining what behavior or risk each task addresses.
- "Run the tests" is not executable verification; a cold-start agent needs the exact command and expected output.
- "The notifier" depends on planning-conversation memory. The task does not state the interface, payload, first action, or test assertion.
- The structure is present, but the document does not transfer enough knowledge for execution.

---

## Anti-Pattern: Horizontal Slicing

The webhook feature planned as layers instead of vertical behavior:

```markdown
### Task 1: Create all data structures
- [ ] Define `WebhookConfig`
- [ ] Define `WebhookPayload`
- [ ] Define `RetryPolicy`

### Task 2: Create all infrastructure
- [ ] Set up HTTP client wrapper
- [ ] Implement retry logic
- [ ] Add config schema

### Task 3: Create all business logic
- [ ] Implement `WebhookNotifier`
- [ ] Wire into `job_runner.py`

### Task 4: Test everything
- [ ] Write all tests
- [ ] Run integration test
```

**Why this fails**
- Task 1 delivers no user-visible behavior and resolves no named uncertainty.
- Task 2 builds infrastructure before proving the completion event can supply a useful payload.
- The first end-to-end verification appears at the end, where wrong assumptions are most expensive to fix.
- A vertical plan would first prove one completed job sends one webhook, then add config and retry behavior.
