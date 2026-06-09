# Large: Plugin System

**Situation:** Add a plugin system so users can drop custom processors into a directory. Plugins are discovered, validated, sandboxed, and run as part of the data pipeline.

**Assessment**
- **Size:** Large. Introduces a new architecture concept and affects pipeline execution, configuration, CLI behavior, plugin API, validation, and sandboxing.
- **Nature:** Architecture change. The plugin interface, loading model, sandboxing boundary, and CLI/config contract must be decided before broad implementation.
- **Current state:** `pipeline/runner.py` calls hardcoded processors in sequence; `config.yaml` stores configuration; no dynamic loading exists.
- **Done state:** A sample plugin is discovered, validated, sandboxed, enabled through config/CLI, and executed by the pipeline.

## Human Review Section

> **Intent:** Let users extend the data pipeline with custom processors without changing core pipeline code.
>
> **Summary (Layer 1)**
> - **Goal:** Add a user-extensible plugin system for data processors.
> - **Why:** Users need custom processing behavior without forking or editing the pipeline.
> - **End state:** Users can place plugin files in a directory, enable valid plugins, and run them safely in the data pipeline.
> - **Impact scope:** Pipeline execution, plugin infrastructure, configuration, and CLI controls.
> - **Size:** Large.
>
> **Summary (Layer 2 - Approach)**
> Define the plugin contract first, prove it with one hardcoded no-op plugin, then resolve sandboxing before detailing discovery and CLI behavior. Discovery and CLI stay goal-level until sandboxing reveals whether plugins can be inspected in-process.
>
> **Change Snapshot**
> - Valid plugin enabled: it runs in the pipeline sandbox.
> - Invalid or unsafe plugin: it is rejected before pipeline execution.
> - No plugins configured: existing pipeline processing remains unchanged.
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

## Execution Plan

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

## Task 1: Plugin Interface Contract

Defines the boundary before building either side of it.

Files: new interface module and `NoOpPlugin` in `plugins/`; tests in `tests/plugins/`.

- [ ] Define plugin interface with `process(data) -> data` and `validate() -> bool`.
- [ ] Define plugin metadata fields: name, version, author, description.
- [ ] Add a no-op plugin that returns data unchanged for testing.
- [ ] Test interface conformance and metadata access.
- [ ] Run `pytest tests/plugins/ -v` -> PASS.
- [ ] Commit point: plugin contract is defined and tested.

## Task 2: Pipeline Walking Skeleton

Proves the contract works through real pipeline execution before dynamic loading exists.

Files: `pipeline/runner.py`; integration tests in `tests/plugins/`.

- [ ] Allow the pipeline runner to accept plugin instances.
- [ ] Wire the no-op plugin from Task 1 into the pipeline.
- [ ] Test that pipeline data passes through the plugin correctly.
- [ ] Run `pytest tests/ -v` -> PASS.
- [ ] Commit point: one hardcoded plugin runs in the pipeline.

## Task 3: Sandboxing Spike and Implementation

Resolves the highest-risk architectural unknown before discovery and CLI are designed in detail.

Files: sandboxing module and tests in `plugins/`.

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

Files: `cli/plugins.py`; `config.yaml`; CLI tests.

Success signal: `pytest tests/cli/test_plugins.py -v` -> PASS, and enabled plugins are the only ones run by the pipeline.

**Final verification:** Place a sample plugin in the plugin directory, run the pipeline, and confirm the plugin is discovered, validated, sandboxed, enabled, and executed.

## Execution Log Example

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
