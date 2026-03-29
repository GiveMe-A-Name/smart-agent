# Writing Plans — Examples

Each example shows: situation assessment → decomposition strategy → plan at the right depth.

---

## Tiny: Bug Fix

**Situation:** API returns 404 when an expired token is used, but should return 401.

**Assessment:**
- Size: Tiny — one condition check in one function, no interface change
- Nature: Bug fix — root cause identified: `auth_middleware.py:47` checks `token.exists()` before `token.is_valid()`, returns 404 on the `exists()` path
- Current state: `authenticate()` in `auth_middleware.py` handles all token errors; no other callers affected

**Plan:**

Root cause: `auth_middleware.py:47` returns 404 for missing tokens but the same 404 is also returned for expired tokens because expiry is checked after existence. Fix: check expiry on the token before returning 404.

Change `auth_middleware.py:47`:
```python
# Before
if not token:
    return Response(status=404)

# After
if not token:
    return Response(status=404)
if not token.is_valid():
    return Response(status=401)
```

Verify: `pytest tests/test_auth.py::test_expired_token_returns_401 -v` → PASS

---

## Small: Add a Feature

**Situation:** CLI `export` command needs a `--dry-run` flag that prints what would be exported without writing files.

**Assessment:**
- Size: Small — touches 3 files, stays in one domain (CLI layer), existing arg-parsing pattern applies
- Nature: Feature — `export.py` and its test already exist, just extending behavior
- Current state: `cli/export.py` uses `argparse`, writes via `FileWriter`; `FileWriter` has a `write()` method that can be conditionally skipped

**Decomposition strategy:** Vertical slicing — each step delivers testable behavior.

**Plan:**

Goal: Add `--dry-run` to `export` command — prints output paths without writing.

Files:
- `cli/export.py` — add `--dry-run` arg, pass flag to writer
- `core/file_writer.py` — accept `dry_run=False` param in `write()`
- `tests/cli/test_export.py` — add test for `--dry-run` output

Steps (each leaves tests green):
1. Add `parser.add_argument('--dry-run', action='store_true')` in `export.py:23`, pass `dry_run=args.dry_run` to `FileWriter` constructor. In `file_writer.py:write()`: if `dry_run`, print path and return without writing.
2. Add test: mock `FileWriter.write`, assert not called when `--dry-run` passed, assert paths printed to stdout.

Verify: `python cli/export.py --dry-run output/` prints paths, no files created; `pytest tests/cli/test_export.py -v` PASS

---

## Medium: New Component (Vertical Slicing + Risk-First)

**Situation:** Add a webhook notification system — when a job completes, POST to a user-configured URL with the result.

**Assessment:**
- Size: Medium — introduces a new concept (webhooks), crosses job execution and config domains, needs new test design
- Nature: Feature — nothing like this exists; need to decide how webhook config is stored and how retries work
- Current state: `job_runner.py` fires `on_complete` event; config is stored in `config.yaml`; no HTTP client currently in use

**Decomposition strategy:**
- **Walking skeleton first** — get the simplest POST working end-to-end before adding config, retries, or error handling
- **Risk-first ordering** — the riskiest part is integrating with `job_runner.py`'s event system, so do that first; retry logic and config are well-understood and can come later
- **Vertical slicing** — each task delivers a new testable behavior, not a layer

**Risk analysis:** The main uncertainty is whether `on_complete` provides enough context to construct the webhook payload. We find out in Task 1.

**File map:**
- Create `notifications/webhook.py` — `WebhookNotifier` class
- Create `tests/notifications/test_webhook.py` — unit + integration tests
- Modify `config.yaml` schema — add `webhooks:` section
- Modify `job_runner.py:on_complete` — wire up `WebhookNotifier`

### Task 1: Walking Skeleton — Hardcoded webhook fires on job completion
*Proves the approach works end-to-end. Resolves the main risk.*

Files: Create `notifications/webhook.py`, `tests/notifications/test_webhook.py`; Modify `job_runner.py`

- [ ] Create `WebhookNotifier` with a `notify(job_result)` method that POSTs to a hardcoded URL using `httpx`
- [ ] Write test: mock HTTP, assert POST is called with correct payload structure
- [ ] Wire into `job_runner.py:on_complete` — instantiate `WebhookNotifier` and call `notify()`
- [ ] Run: `pytest tests/notifications/test_webhook.py -v` → PASS
- [ ] Commit point: tests green, webhook fires with hardcoded URL

### Task 2: Make it configurable
*Known-how-to-do work — low risk.*

Files: Modify `config.yaml`, `notifications/webhook.py`, `job_runner.py`

- [ ] Add `webhooks:` section to config schema with `url` and `secret` fields
- [ ] Modify `WebhookNotifier` to accept URL from config instead of hardcoded
- [ ] In `job_runner.py`: read config, only instantiate `WebhookNotifier` if `webhooks.url` is set
- [ ] Add test: no webhook call when config is absent
- [ ] Run: `pytest tests/ -v` → PASS
- [ ] Commit point: tests green, webhook is config-driven

### Task 3: Add retry logic
*Routine work, well-understood patterns.*

Files: Modify `notifications/webhook.py`, `tests/notifications/test_webhook.py`

- [ ] Add retry logic: up to 3 retries with exponential backoff on 5xx, give up immediately on 4xx
- [ ] Write tests: retry on 503, no retry on 400, success on 2nd attempt
- [ ] Run: `pytest tests/notifications/test_webhook.py -v` → PASS
- [ ] Commit point: tests green, retry logic complete

**Final verification:** Run a test job with webhook URL pointed at `localhost` echo server, confirm POST received with correct payload.

---

## Large: Multi-Component System (Full Strategy)

**Situation:** Add a plugin system that lets users write custom processors. Processors receive data from the pipeline, transform it, and return results. Plugins are loaded from a directory, auto-discovered, validated, and sandboxed.

**Assessment:**
- Size: Large — new architecture concept, affects pipeline, config, CLI, and introduces plugin API contract
- Nature: Architecture — design decisions about plugin interface, discovery, loading, and sandboxing
- Current state: Pipeline is hardcoded in `pipeline/runner.py`, calls each processor in sequence. Config is `config.yaml`. No dynamic loading exists.

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

Files: Create `plugins/interface.py`, `plugins/__init__.py`, `tests/plugins/test_interface.py`

- [ ] Define `PluginBase` ABC with `process(data: PipelineData) -> PipelineData` and `validate() -> bool`
- [ ] Define `PluginMetadata` dataclass: name, version, author, description
- [ ] Write a concrete `NoOpPlugin` that passes data through unchanged (useful for testing)
- [ ] Write tests: `NoOpPlugin` conforms to interface, metadata is accessible
- [ ] Run: `pytest tests/plugins/ -v` → PASS
- [ ] Commit point: plugin contract is defined and tested

### Task 2: Walking Skeleton — Hardcoded plugin runs in pipeline

*Proves the approach works end-to-end without any dynamic loading.*

Files: Modify `pipeline/runner.py`; Create `tests/plugins/test_pipeline_integration.py`

- [ ] Modify `pipeline/runner.py` to accept a list of `PluginBase` instances in addition to hardcoded processors
- [ ] Wire `NoOpPlugin` into the pipeline as a test
- [ ] Write integration test: pipeline runs with `NoOpPlugin`, data passes through correctly
- [ ] Run: `pytest tests/ -v` → PASS
- [ ] Commit point: pipeline supports plugins, one hardcoded plugin works

### Task 3: Sandboxing Spike + Implementation

*Risk-first: this is the biggest unknown. Validate approach before building on it.*

Files: Create `plugins/sandbox.py`, `tests/plugins/test_sandbox.py`

- [ ] Research and implement sandboxing approach (subprocess isolation vs. restricted globals)
- [ ] Create `SandboxedPluginRunner` that executes a plugin's `process()` in isolation
- [ ] Write tests: plugin cannot access filesystem, network timeout is enforced, crash doesn't kill host
- [ ] Run: `pytest tests/plugins/test_sandbox.py -v` → PASS
- [ ] Commit point: sandboxing works, constraints are known
- [ ] **After this task**: refine Tasks 4-5 based on what sandboxing constraints revealed

### Task 4: Auto-Discovery and Validation (refine after Task 3)

Goal: Load plugins from a configured directory, validate them against the interface, register valid plugins.

Files: Create `plugins/discovery.py`, `plugins/validator.py`; Modify `config.yaml`

*Detail to be filled after Task 3 is complete.*

### Task 5: CLI and Config Integration (refine after Task 3, parallel with Task 4)

Goal: Add CLI commands to list/enable/disable plugins. Add config section for plugin directory and enabled plugins.

Files: Modify `cli/plugins.py`, `config.yaml`

*Detail to be filled after Task 3 is complete.*

**Final verification:** Place a sample plugin in the plugins directory, run the pipeline, confirm it's auto-discovered, validated, sandboxed, and executed correctly.

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
