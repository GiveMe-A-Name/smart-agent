# Large: Plugin System

## Decision Brief

**Intent:** Let users extend the data pipeline with custom processors without modifying or forking core pipeline code.

**Outcome:** Users can discover, enable, validate, and safely run compatible plugins while pipelines without plugins keep their existing behavior.

**Scope:** Plugin contracts, isolated execution, discovery, configuration, CLI controls, and pipeline integration; host-process safety takes priority over extension flexibility.

**Visible Changes:**
- Valid enabled plugin: run it through the isolated plugin boundary.
- Invalid or unsafe plugin: reject it before it can affect pipeline execution.
- No plugin configured: preserve current pipeline processing.

**Approval Needed:** Approve an explicit plugin contract and an isolation-first architecture; select the concrete sandbox mechanism only after an early spike compares subprocess isolation with restricted in-process execution.

**Primary Risk:** The sandbox mechanism determines whether discovery and validation may inspect plugin code in-process, so their detailed design must remain provisional until the spike completes.

## Design Review

The pipeline depends only on a host-owned plugin runner. Untrusted plugin code executes beyond a sandbox boundary; discovery and CLI consume metadata and registration results rather than importing or invoking plugins directly.

```text
pipeline runner -> plugin runner -> sandbox boundary -> plugin code
CLI/config -> discovery and validation -> plugin metadata/registration
```

The plugin contract exposes metadata, input validation, and processing. Execution returns success, validation failure, timeout, sandbox violation, or plugin crash so the host can apply an explicit failure policy without leaking plugin exceptions into core pipeline control flow.

The material alternative is direct in-process loading. It is rejected as the default shape because it gives discovery and pipeline execution access to untrusted imports before isolation is established. The spike may choose subprocess or restricted in-process isolation, but it must preserve the one-way dependency and prevent the pipeline runner from discovering, importing, or sandboxing plugins itself.

**Assessment**
- **Size:** Large. Introduces a new architecture concept and affects pipeline execution, configuration, CLI behavior, plugin API, validation, and sandboxing.
- **Nature:** Architecture change. The plugin interface, loading model, sandboxing boundary, and CLI/config contract must be decided before broad implementation.
- **Current state:** `pipeline/runner.py` calls hardcoded processors in sequence; `config.yaml` stores configuration; no dynamic loading exists.
- **Done state:** A sample plugin is discovered, validated, sandboxed, enabled through config/CLI, and executed by the pipeline.

## Execution Plan

**Decomposition strategy**
- **Contract first:** define the interface before building either side.
- **Walking skeleton:** prove one plugin can run through the pipeline before dynamic loading.
- **Risk first:** sandboxing is the biggest unknown and must be settled before discovery and CLI details.
- **Progressive refinement:** Tasks 1-3 are detailed; Tasks 4-5 stay goal-level until Task 3 completes.

**Execution risks and plan-internal responses**
1. **Risk:** The executable plugin contract may couple static compatibility checks to plugin imports, exposing untrusted code before isolation. **Contingency:** keep compatibility data in host-readable metadata if doing so preserves the approved contract and isolation boundary.
2. **Risk:** Restricted in-process execution may fail the filesystem, timeout, or crash-containment checks. **Contingency:** use subprocess isolation; both mechanisms are inside the approved sandbox boundary.
3. **Risk:** Task 4's module split depends on the chosen sandbox, so detailed files selected now may be wrong. **Revision trigger:** keep it goal-level and refine its files and handoff after Task 3 without moving discovery or imports into the pipeline runner.

**Explicit exclusions**
- No plugin marketplace, dependency installer, or remote plugin distribution.
- No hot reload of plugins during a running pipeline.
- No direct plugin access to host configuration or pipeline internals.

**Dependency graph**

```text
Task 1: Plugin interface
  -> Task 2: Pipeline walking skeleton
  -> Task 3: Sandboxing
    -> Task 4: Discovery and validation
    -> Task 5: CLI and config controls
```

Tasks 4 and 5 may run in parallel only after Task 3 defines the sandboxing contract and their file ownership is separated.

**Stop signals**
- If the plugin interface cannot support both validation and processing without importing untrusted code in-process, pause before changing the contract.
- If implementation evidence requires changing the approved split between pipeline execution, plugin contract, plugin runner, and sandboxing, pause before revising the plan.

**File map**
- `pipeline/runner.py`: call the host-owned plugin runner without discovering or importing plugins.
- `config.yaml`: store plugin enablement after sandbox and discovery constraints are known.
- `plugins/`: own the contract, runner, sandbox, discovery, and validation responsibilities.
- `cli/plugins.py`: expose list, enable, and disable controls after registration is stable.
- `tests/plugins/`: cover contract, pipeline integration, sandbox, and discovery behavior.
- `tests/cli/test_plugins.py`: prove CLI controls affect the enabled plugin set.

## Task 1: Plugin Interface Contract

Defines the boundary before building either side of it.

Files: new `plugins/interface.py`; new `plugins/noop.py`; new `tests/plugins/test_interface.py`.

- [ ] Define plugin interface with `process(data) -> data` and `validate() -> ValidationResult`.
- [ ] Define plugin metadata fields: name, version, author, description.
- [ ] Add a no-op plugin that returns data unchanged for testing.
- [ ] Test interface conformance and metadata access.
- [ ] Run `pytest tests/plugins/ -v` -> PASS.
- [ ] Commit point: plugin contract is defined and tested.

## Task 2: Pipeline Walking Skeleton

Proves the contract works through real pipeline execution before dynamic loading exists.

Depends on: Task 1's plugin interface and `NoOpPlugin`.

Files: `pipeline/runner.py`; new `plugins/runner.py`; new `tests/plugins/test_pipeline.py`.

- [ ] Allow the pipeline runner to execute plugins through the `plugins/runner.py` facade.
- [ ] Wire the no-op plugin from Task 1 through that facade.
- [ ] Test that pipeline data passes through the plugin correctly.
- [ ] Run `pytest tests/ -v` -> PASS.
- [ ] Commit point: one hardcoded plugin runs in the pipeline.

## Task 3: Sandboxing Spike and Implementation

Resolves the highest-risk architectural unknown before discovery and CLI are designed in detail.

Depends on: Task 2's host-side runner facade.

Files: `plugins/runner.py`; new `plugins/sandbox.py`; new `tests/plugins/test_sandbox.py`.

- [ ] Compare subprocess isolation and restricted globals against filesystem access, timeout, and host-process safety.
- [ ] Implement the chosen sandboxed runner.
- [ ] Test that filesystem access is blocked, timeout is enforced, and plugin crashes do not kill the host process.
- [ ] Run `pytest tests/plugins/ -v` -> PASS.
- [ ] Commit point: sandboxing works and its constraints are documented in the execution log.
- [ ] Revision trigger: after this task, refine Tasks 4-5 based on the sandboxing result.

## Task 4: Discovery and Validation (Goal-Level)

Goal: Load plugin candidates from a configured directory, validate them against the interface, and register only valid plugins.

Depends on: Task 3 sandboxing constraints.

Files: discovery/validation modules in `plugins/`; `config.yaml`. Exact module split depends on Task 3.

Success signal: `pytest tests/plugins/test_discovery.py -v` -> PASS, with valid plugins registered and invalid plugins rejected.

## Task 5: CLI and Config Controls (Goal-Level)

Goal: Let users list, enable, and disable discovered plugins.

Depends on: Task 3 sandboxing constraints and the registration shape from Task 4.

Files: `cli/plugins.py`; `config.yaml`; `tests/cli/test_plugins.py`.

Success signal: `pytest tests/cli/test_plugins.py -v` -> PASS, and enabled plugins are the only ones run by the pipeline.

**Final verification:** Run `pytest tests/plugins/ tests/cli/test_plugins.py -v && python scripts/smoke_plugin_pipeline.py fixtures/plugins/noop_plugin.py` -> PASS and prints `sample plugin executed`.

## Execution Log

*(append entries here as each task completes - format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*
