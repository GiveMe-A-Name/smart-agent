# Example: Progressive Execution Plan (Plugin System)

Compact reference for a medium/large plan written to a temporary file outside git tracking. Near tasks are concrete; later tasks stay goal-level until the sandboxing spike resolves their constraints.

## Assessment

- **Outcome:** Users can discover, validate, enable, and run isolated plugins in the data pipeline; pipelines without plugins keep current behavior.
- **Current state:** `pipeline/runner.py` executes hardcoded processors in sequence; `config.yaml` stores pipeline config; no dynamic plugin loading or sandbox boundary exists.
- **Constraints:** `pipeline/runner.py` must depend only on a host-owned plugin runner facade and never import untrusted plugin code directly. Host-process safety takes priority over extension flexibility.
- **Final verification:** `pytest tests/plugins/ tests/cli/test_plugins.py -v && python scripts/smoke_plugin_pipeline.py fixtures/plugins/noop_plugin.py` -> PASS and prints `sample plugin executed`.

## Exclusions & Escalation

- **Exclusions:** No plugin marketplace or dependency installer; no hot reload during a running pipeline; no direct plugin access to host config or pipeline internals.
- **Escalate if:**
  - Validation cannot inspect plugin compatibility without importing untrusted code in-process.
  - Neither subprocess isolation nor restricted in-process execution can meet filesystem, timeout, and crash-containment constraints without adding a new external dependency.

## Tasks

### Task 1: Plugin Interface Contract

Files: new `plugins/interface.py`, new `plugins/noop.py`, new `tests/plugins/test_interface.py`

- [ ] Define the plugin contract: metadata (`name`, `version`, `author`, `description`), `validate() -> ValidationResult`, and `process(data) -> data`, plus explicit error/result categories (success, validation failure, timeout, sandbox violation, crash).
- [ ] Add a `NoOpPlugin` that returns input data unchanged for testing.
- [ ] Verify: `pytest tests/plugins/test_interface.py -v` -> PASS.

### Task 2: Pipeline Walking Skeleton

Depends on: Task 1

Files: `pipeline/runner.py`, new `plugins/runner.py`, new `tests/plugins/test_pipeline.py`

- [ ] Introduce a host-side `plugins/runner.py` facade and wire `pipeline/runner.py` to execute enabled plugins through it, preserving existing behavior when no plugins are enabled.
- [ ] Run `NoOpPlugin` end-to-end through the pipeline facade.
- [ ] Verify: `pytest tests/ -v` -> PASS.

### Task 3: Sandboxing Spike & Implementation

Depends on: Task 2

Files: `plugins/runner.py`, new `plugins/sandbox.py`, new `tests/plugins/test_sandbox.py`

- [ ] Compare subprocess isolation vs. restricted in-process execution against filesystem blocking, timeout enforcement, and crash containment; pick the viable mechanism.
- [ ] Integrate the chosen sandbox into `plugins/runner.py` so plugin crashes/timeouts return structured failures without killing the host process.
- [ ] Verify: `pytest tests/plugins/test_sandbox.py tests/plugins/test_pipeline.py -v` -> PASS, covering filesystem block, timeout, and crash isolation.
- [ ] **Revision trigger:** Update Tasks 4–5 in place once the chosen sandbox determines how plugin metadata and validation can run safely.

### Task 4: Discovery & Validation (Goal-Level)

Depends on: Task 3

- **Goal:** Load plugin candidates from a configured directory, validate against the interface under Task 3's sandbox constraints, and register only valid plugins.
- **Target area:** `plugins/`, `config.yaml`, `tests/plugins/test_discovery.py`
- **Success signal:** `pytest tests/plugins/test_discovery.py -v` -> PASS (valid plugins registered; invalid/unsafe plugins rejected).

### Task 5: CLI & Config Controls (Goal-Level)

Depends on: Tasks 3–4

- **Goal:** Expose CLI and config controls to list, enable, and disable discovered plugins so only enabled plugins run in the pipeline.
- **Target area:** `cli/plugins.py`, `config.yaml`, `tests/cli/test_plugins.py`
- **Success signal:** `pytest tests/cli/test_plugins.py -v` -> PASS.
