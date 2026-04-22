# Writing Plans — Examples

Each example shows: situation assessment → decomposition strategy → plan at the right depth.

---

## Tiny: Bug Fix

**Situation:** API returns 404 when an expired token is used, but should return 401.

**Assessment:**
- Size: Tiny — one condition check in one function, no interface change
- Nature: Bug fix — root cause identified: `auth_middleware.py:47` checks `token.exists()` before `token.is_valid()`, returns 404 on the `exists()` path
- Current state: `authenticate()` in `auth_middleware.py` handles all token errors; no other callers affected

> **Intent:** Fix the authentication bug where expired tokens show users a "not found" error instead of prompting them to log in again.
>
> **Summary (Layer 1)**
> - **Goal:** Fix expired tokens returning 404 instead of 401
> - **Why:** Users see "not found" errors when their tokens expire, confusing them into thinking the endpoint doesn't exist
> - **Expected outcome:** Expired tokens correctly return 401 Unauthorized
> - **Impact scope:** Authentication middleware only
> - **Size:** Tiny

**Plan:**

Root cause: `auth_middleware.py:47` checks token existence before validity — expired tokens fall through the existence check and return 404 instead of 401. Fix: add a validity check at `auth_middleware.py:47` so expired tokens return 401.

Verify: `pytest tests/test_auth.py::test_expired_token_returns_401 -v` → PASS

---

## Small: Add a Feature

**Situation:** CLI `export` command needs a `--dry-run` flag that prints what would be exported without writing files.

**Assessment:**
- Size: Small — touches 3 files, stays in one domain (CLI layer), existing arg-parsing pattern applies
- Nature: Feature — `export.py` and its test already exist, just extending behavior
- Current state: `cli/export.py` uses `argparse`, writes via `FileWriter`; `FileWriter` has a `write()` method that can be conditionally skipped

> **Intent:** Give CLI users a way to preview what the export command will do before actually writing any files.
>
> **Summary (Layer 1)**
> - **Goal:** Add `--dry-run` flag to CLI `export` command
> - **Why:** Users want to preview what files would be exported before actually writing them
> - **Expected outcome:** Running `export --dry-run` prints output paths without creating any files
> - **Impact scope:** CLI export command and file writer
> - **Size:** Small

**Decomposition strategy:** Vertical slicing — each step delivers testable behavior.

**Plan:**

Goal: Add `--dry-run` to `export` command — prints output paths without writing.

Files:
- `cli/export.py` — add `--dry-run` arg, pass flag to writer
- `core/file_writer.py` — accept `dry_run=False` param in `write()`
- `tests/cli/test_export.py` — add test for `--dry-run` output

Steps (each leaves tests green):
1. Add `--dry-run` flag to `export.py` arg parser; pass flag through to `FileWriter`. In `file_writer.py:write()`: when dry-run is active, print the output path and return without writing.
2. Add test: assert `FileWriter.write` is not called when `--dry-run` is passed; assert output paths are printed to stdout.

Verify: `python cli/export.py --dry-run output/` prints paths, no files created; `pytest tests/cli/test_export.py -v` PASS

---

## Medium: New Component (Vertical Slicing + Risk-First)

**Situation:** Add a webhook notification system — when a job completes, POST to a user-configured URL with the result.

**Assessment:**
- Size: Medium — introduces a new concept (webhooks), crosses job execution and config domains, needs new test design
- Nature: Feature — nothing like this exists; need to decide how webhook config is stored and how retries work
- Current state: `job_runner.py` fires `on_complete` event; config is stored in `config.yaml`; no HTTP client currently in use

> **Intent:** Let users receive real-time notifications in their own systems when jobs complete, without having to poll.
>
> **Summary (Layer 1)**
> - **Goal:** Add webhook notifications when jobs complete
> - **Why:** Users need real-time notification of job results in their own systems without polling
> - **Expected outcome:** When a job finishes, the system POSTs the result to a user-configured URL with retry on failure
> - **Impact scope:** Job execution system, configuration, new notification module
> - **Size:** Medium
>
> **Summary (Layer 2 — Approach)**
>
> Build incrementally starting with the riskiest part — integrating with the job completion event system. Three steps:
> 1. Get a hardcoded webhook firing end-to-end on job completion (proves the approach works)
> 2. Make the webhook URL configurable (straightforward, low risk)
> 3. Add retry with exponential backoff (well-understood pattern)
>
> Main risk: the job completion event might not carry enough context to construct a useful payload. We find out in step 1 before investing in config and retry logic.

**Decomposition strategy:**
- **Walking skeleton first** — get the simplest POST working end-to-end before adding config, retries, or error handling
- **Risk-first ordering** — the riskiest part is integrating with `job_runner.py`'s event system, so do that first; retry logic and config are well-understood and can come later
- **Vertical slicing** — each task delivers a new testable behavior, not a layer

**Risk analysis:** The main uncertainty is whether `on_complete` provides enough context to construct the webhook payload. We find out in Task 1.

**Explicit exclusions:**
- Not adding webhook signing/authentication — URLs are assumed internal
- Not supporting multiple webhook targets per job
- Not building a webhook management UI — URL is config-only

**Conflict priority:** When feature completeness conflicts with schedule, prefer a working subset over a partial implementation of everything. Retry logic (Task 3) is lower priority than a working configurable webhook (Tasks 1–2).

**Stop signals:**
- If `job_runner.py`'s `on_complete` event doesn't carry enough data to build a useful payload → pause and ask before designing a workaround
- If retry logic requires introducing a new external dependency → pause and confirm before adding it

**File map** *(design sketch — names may change during execution)*:
- Create webhook notifier in `notifications/` — likely `notifications/webhook.py`
- Create tests in `tests/notifications/`
- Modify `config.yaml` schema — add `webhooks:` section
- Modify `job_runner.py:on_complete` — wire up the notifier

### Task 1: Walking Skeleton — Hardcoded webhook fires on job completion
*Proves the approach works end-to-end. Resolves the main risk.*

Files: Create webhook notifier module in `notifications/` and its tests in `tests/notifications/`; Modify `job_runner.py`

- [ ] Create a notifier with a `notify(job_result)` method that POSTs to a hardcoded URL
- [ ] Write test: mock HTTP, assert POST is called with correct payload structure
- [ ] Wire into `job_runner.py:on_complete`
- [ ] Run: `pytest tests/notifications/ -v` → PASS
- [ ] Commit point: tests green, webhook fires with hardcoded URL

### Task 2: Make it configurable
*Known-how-to-do work — low risk.*

Files: Modify `config.yaml`, the notifier created in Task 1, `job_runner.py`

- [ ] Add `webhooks:` section to config schema with `url` and `secret` fields
- [ ] Notifier accepts URL from config instead of hardcoded
- [ ] In `job_runner.py`: read config, only instantiate notifier if `webhooks.url` is set
- [ ] Add test: no webhook call when config is absent
- [ ] Run: `pytest tests/ -v` → PASS
- [ ] Commit point: tests green, webhook is config-driven

### Task 3: Add retry logic
*Routine work, well-understood patterns.*

Files: Modify the notifier and its tests (created in Task 1)

- [ ] Add retry with exponential backoff on 5xx, give up immediately on 4xx
- [ ] Write tests: retry on 503, no retry on 400, success on 2nd attempt
- [ ] Run: `pytest tests/notifications/ -v` → PASS
- [ ] Commit point: tests green, retry logic complete

**Final verification:** Run a test job with webhook URL pointed at `localhost` echo server, confirm POST received with correct payload.

---

## Execution Log

*(append entries here as each task completes — format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*

---

### After execution (example)

[2026-04-02] Task 1: complete — walking skeleton works end-to-end. `on_complete` passes a
  `JobResult` object with `job_id`, `status`, and `output`; payload mapping is straightforward
  and no schema changes are needed.

[2026-04-02] Task 2: complete — `WebhookNotifier` now reads URL from `config.yaml webhooks.url`;
  instantiation is skipped when config is absent, so no accidental webhook calls in
  environments without config.

[2026-04-03] Task 3: failed — `pytest tests/notifications/test_webhook.py::test_retry_on_503 FAILED`
  → RuntimeError: no running event loop — `httpx.AsyncClient.send()` called outside asyncio
  event loop in the test environment. Switched to `tenacity.retry` wrapping synchronous
  `httpx.post` instead of httpx's built-in async retry. `pytest tests/notifications/test_webhook.py -v`
  → PASS. Task 3: complete.

---

## Large: Multi-Component System (Full Strategy)

**Situation:** Add a plugin system that lets users write custom processors. Processors receive data from the pipeline, transform it, and return results. Plugins are loaded from a directory, auto-discovered, validated, and sandboxed.

**Assessment:**
- Size: Large — new architecture concept, affects pipeline, config, CLI, and introduces plugin API contract
- Nature: Architecture — design decisions about plugin interface, discovery, loading, and sandboxing
- Current state: Pipeline is hardcoded in `pipeline/runner.py`, calls each processor in sequence. Config is `config.yaml`. No dynamic loading exists.

> **Intent:** Let users extend the data pipeline with their own custom processing logic by dropping plugin files into a directory, without touching core pipeline code.
>
> **Summary (Layer 1)**
> - **Goal:** Add a user-extensible plugin system for custom data processors
> - **Why:** Users need to add custom processing logic without modifying core pipeline code
> - **Expected outcome:** Users drop plugin files into a directory; the system auto-discovers, validates, sandboxes, and runs them as part of the pipeline. CLI commands let users list/enable/disable plugins.
> - **Impact scope:** Data pipeline, configuration system, CLI, new plugin infrastructure
> - **Size:** Large
>
> **Summary (Layer 2 — Approach)**
>
> Contract-first design: define the plugin interface before building anything on top of it. Then prove it works with a hardcoded plugin through the pipeline (walking skeleton). Tackle sandboxing early — it's the biggest unknown and affects everything downstream.
>
> Key design decisions:
> - Plugins implement a standard interface (`process` + `validate`) with metadata
> - Sandboxing approach (subprocess vs. restricted globals) determined by early spike — later tasks adapt based on findings
> - Auto-discovery and CLI commands planned at goal-level, detailed after sandboxing constraints are known
>
> Main risks: (1) getting the plugin interface contract wrong forces rework on everything, (2) sandboxing may impose constraints that change how discovery and CLI work.

**Decomposition strategy:**
- **Contract-first** — define the plugin interface before building discovery or loading. This is the foundation everything else depends on.
- **Walking skeleton** — get one hardcoded "fake plugin" working through the pipeline before adding auto-discovery
- **Risk-first ordering** — sandboxing is the biggest unknown; validate it early (Task 3) before building everything else on top
- **Progressive refinement** — Tasks 1-3 fully detailed; Tasks 4-5 at goal level (will refine after Task 3 reveals sandboxing constraints)

**Risk analysis:**
1. Plugin interface design — if the contract is wrong, everything built on it needs rework → Task 1
2. Sandboxing approach — is `subprocess` enough? Do we need `seccomp`? → Task 3 (early)
3. Plugin validation — can we validate plugins without loading them? → Task 4 (after sandboxing is settled)

**Dependency graph:**
```
Task 1 (Plugin Interface) 
  → Task 2 (Walking Skeleton)
  → Task 3 (Sandboxing) 
    → Task 4 (Auto-Discovery + Validation)
    → Task 5 (CLI + Config)
```
Tasks 4 and 5 can be done in parallel after Task 3.

### Task 1: Define Plugin Interface Contract

*Contract-first: define the boundary before building either side. This is the foundation.*

Files: Create interface module and `NoOpPlugin` in `plugins/`; create tests in `tests/plugins/`

- [ ] Define plugin ABC with `process(data) -> data` and `validate() -> bool`
- [ ] Define plugin metadata structure: name, version, author, description
- [ ] Write a concrete no-op plugin that passes data through unchanged (useful for testing)
- [ ] Write tests: no-op plugin conforms to interface, metadata is accessible
- [ ] Run: `pytest tests/plugins/ -v` → PASS
- [ ] Commit point: plugin contract is defined and tested

### Task 2: Walking Skeleton — Hardcoded plugin runs in pipeline

*Proves the approach works end-to-end without any dynamic loading.*

Files: Modify `pipeline/runner.py`; create integration test in `tests/plugins/`

- [ ] Pipeline runner accepts plugin instances in addition to hardcoded processors
- [ ] Wire no-op plugin (from Task 1) into the pipeline
- [ ] Integration test: pipeline runs with plugin, data passes through correctly
- [ ] Run: `pytest tests/ -v` → PASS
- [ ] Commit point: pipeline supports plugins, one hardcoded plugin works

### Task 3: Sandboxing Spike + Implementation

*Risk-first: this is the biggest unknown. Validate approach before building on it.*

Files: Create sandboxing module and its tests in `plugins/`

- [ ] Research and implement sandboxing approach (subprocess isolation vs. restricted globals)
- [ ] Sandboxed runner executes a plugin's `process()` in isolation
- [ ] Tests: plugin cannot access filesystem, network timeout is enforced, crash doesn't kill host
- [ ] Run: `pytest tests/plugins/ -v` → PASS
- [ ] Commit point: sandboxing works, constraints are known
- [ ] **After this task**: refine Tasks 4-5 based on what sandboxing constraints revealed

### Task 4: Auto-Discovery and Validation *(goal-level — depends on Task 3)*

Goal: Load plugins from a configured directory, validate against the interface, register valid plugins.

Files: Create discovery and validation modules in `plugins/` (structure depends on sandboxing approach from Task 3); Modify `config.yaml`

*Approach depends on sandboxing constraints revealed in Task 3. Will be detailed after Task 3 completes.*

- [ ] Plugins are discovered from a configured directory without loading plugin code at discovery time
- [ ] Invalid plugins (wrong interface, timeout, crash) are rejected before registration
- [ ] Valid plugins are registered in a form the pipeline runner can use at runtime
- [ ] Run: `pytest tests/plugins/test_discovery.py -v` → PASS
- [ ] Commit point: discovery and validation work; invalid plugins are rejected

### Task 5: CLI and Config Integration *(goal-level — depends on Task 3; parallel with Task 4)*

Goal: Add CLI commands to list/enable/disable plugins.

Files: Modify `cli/plugins.py`, `config.yaml`

*Interaction model depends on sandboxing constraints from Task 3 — e.g. whether plugins can be inspected in-process.*

- [ ] Users can list discovered plugins with enabled/disabled status
- [ ] Users can enable/disable individual plugins
- [ ] Run: `pytest tests/cli/test_plugins.py -v` → PASS
- [ ] Commit point: CLI controls which plugins are active

**Final verification:** Place a sample plugin in the plugins directory, run the pipeline, confirm it's auto-discovered, validated, sandboxed, and executed correctly.

---

## Execution Log

*(append entries here as each task completes — format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*

---

### After execution (example)

[2026-04-06] Task 1: complete — `PluginBase` ABC, `PluginMetadata`, and `NoOpPlugin` defined.
  `PipelineData` moved to `plugins/interface.py` to avoid circular import — `plugins/` cannot
  import from `pipeline/` at module level. All tasks that modify `pipeline/` should import
  `PipelineData` from `plugins/interface.py`.

[2026-04-06] Task 2: complete — `NoOpPlugin` wired into `pipeline/runner.py`; confirmed plugins
  are called mid-pipeline (not in `on_complete`), data flows through synchronously. Plugin list
  is passed at runner construction time, not per-run.

[2026-04-07] Task 3: failed first attempt — used `RestrictedPython` to block `__import__`,
  `open`, `socket`. `pytest tests/plugins/test_sandbox.py::test_filesystem_access FAILED`
  → plugin accessed filesystem via `importlib.import_module` bypass; `RestrictedPython`
  cannot reliably block all stdlib entry points without patching each one individually.
  Switched to subprocess isolation: `SandboxedPluginRunner` spawns child process via
  `subprocess.Popen`, plugin communicates over JSON stdin/stdout.
  `pytest tests/plugins/test_sandbox.py -v` → PASS. Task 3: complete.

  Plan revision triggered — sandboxing revealed plugins are fully out-of-process and
  stateless from the host's perspective. Updated goal-level Tasks 4-5:
  - Task 4 revised: validation must run inside subprocess, not import class directly;
    discovery reads metadata from a manifest file (e.g. `plugin.json`) without loading
    any plugin code; registration stores `(path, metadata)` tuples, not class references.
  - Task 5 revised: CLI operates on config file only; in-process plugin introspection
    is not possible with subprocess isolation.

[2026-04-08] Task 4: failed — `pytest tests/plugins/test_discovery.py::test_invalid_plugin_rejected FAILED`
  → subprocess hung indefinitely; plugin's `validate()` called `input()`, blocking on stdin.
  Fix: added 5s timeout to `SandboxedPluginRunner.run()`; plugins exceeding timeout are
  rejected as invalid. `pytest tests/plugins/ -v` → PASS. Task 4: complete.
  Files actually created: `plugins/discovery.py` (scan + validate combined — separate
  validator module wasn't needed since validation is one `SandboxedPluginRunner.run()` call).

[2026-04-09] Task 5: complete — added `plugin list / enable / disable` CLI commands;
  config key `plugins.enabled[]` drives which discovered plugins are active at runtime.
  Disabled plugins are still discovered and validated, just not loaded into the pipeline.

---

## Anti-Pattern: Horizontal Slicing (What NOT to do)

Here is the same webhook example from Medium, planned badly with horizontal slicing:

### Task 1: Create all data structures
- [ ] Define `WebhookConfig` dataclass
- [ ] Define `WebhookPayload` dataclass
- [ ] Define `RetryPolicy` dataclass

### Task 2: Create all infrastructure
- [ ] Set up `httpx` client wrapper
- [ ] Implement retry logic
- [ ] Add config schema

### Task 3: Create all business logic
- [ ] Implement `WebhookNotifier`
- [ ] Wire into `job_runner.py`

### Task 4: Test everything
- [ ] Write all tests
- [ ] Run integration test

**Why this is bad:**
- Task 1 delivers nothing verifiable — you can't test data structures in isolation meaningfully
- Task 2 builds infrastructure you can't verify works with real data yet
- You don't discover that `on_complete` doesn't provide enough context until Task 3
- If assumptions were wrong in Task 1, you've wasted all of Tasks 1-2
- The first point where you can see it work is the last task
