# Large: Plugin System

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

**Situation:** Add a plugin system so users can drop custom processors into a directory. Plugins are discovered, validated, sandboxed, and run as part of the data pipeline.

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

**Stop signals**
- If the plugin interface cannot support both validation and processing without importing untrusted code in-process, pause before changing the contract.
- If implementation evidence shows the Concrete Design Sketch cannot be followed, pause before making an architectural divergence.

## Concrete Design Sketch

**Target shape**
- `plugins/interface.py`: stable plugin protocol, metadata shape, and host-visible data contract.
- `plugins/noop_plugin.py`: sample plugin used to prove the contract.
- `plugins/runner.py`: host-side execution facade used by the pipeline runner; Task 2 may call the trusted no-op plugin directly, and Task 3 moves non-core plugin execution behind the sandbox.
- `plugins/sandbox.py`: after Task 3, the only module allowed to execute non-core plugin code.
- `pipeline/runner.py`: orchestrates built-in processors and enabled plugin processors; does not discover or sandbox plugins itself.
- Post-Task-3 provisional slots: discovery/validation modules under `plugins/` and CLI controls under `cli/`; exact filenames and flow are revised after sandboxing is known.

**Boundary surfaces**

```text
Plugin.process(data: PipelineData) -> PipelineData
Plugin.validate() -> ValidationResult

PluginMetadata:
  name
  version
  author
  description

PluginRunner.run(plugin_ref, data) -> PluginRunResult
```

**Intended flow**

```text
pipeline_runner.run(data, config):
  processors = built_in_processors(config)
  plugins = enabled plugin refs from the post-Task-3 registration contract
  for processor in processors + plugins:
    data = plugin_runner.run(processor, data)
  return data

plugin_runner.run(plugin_ref, data):
  if plugin_ref is the trusted no-op plugin during Task 2:
    return plugin_ref.process(data)

  after Task 3:
    execute non-core plugin code through the chosen sandbox boundary
    return PluginRunResult or failed_pipeline_result(reason)

Post-Task-3 invariants:
  discovery produces enabled plugin refs without importing untrusted processors in pipeline_runner
  plugin_runner executes non-core plugin code only through the chosen sandbox boundary
```

**Ownership**
- plugin interface owns the public plugin contract and no loading behavior.
- pipeline runner owns processor ordering only; it does not import arbitrary plugin files.
- plugin runner owns the host-facing execution call shape.
- sandbox owns execution isolation, timeout, crash handling, and filesystem restrictions.
- discovery and CLI ownership are provisional until Task 3 completes; the revised plan must keep discovery out of host-process execution and CLI out of validation/execution.

**Shape to avoid**

```text
pipeline_runner.run(data, config):
  files = list plugin directory
  import each plugin file in-process
  call plugin.process(data)
  catch failures inline
  update config or CLI output
```

## Task 1: Plugin Interface Contract

Defines the boundary before building either side of it.

Files: new interface module and `NoOpPlugin` in `plugins/`; tests in `tests/plugins/`.

- [ ] Define plugin interface with `process(data) -> data` and `validate() -> ValidationResult`.
- [ ] Define plugin metadata fields: name, version, author, description.
- [ ] Add a no-op plugin that returns data unchanged for testing.
- [ ] Test interface conformance and metadata access.
- [ ] Run `pytest tests/plugins/ -v` -> PASS.
- [ ] Commit point: plugin contract is defined and tested.

## Task 2: Pipeline Walking Skeleton

Proves the contract works through real pipeline execution before dynamic loading exists.

Files: `pipeline/runner.py`; `plugins/runner.py`; integration tests in `tests/plugins/`.

- [ ] Allow the pipeline runner to execute plugins through the `plugins/runner.py` facade.
- [ ] Wire the no-op plugin from Task 1 through that facade.
- [ ] Test that pipeline data passes through the plugin correctly.
- [ ] Run `pytest tests/ -v` -> PASS.
- [ ] Commit point: one hardcoded plugin runs in the pipeline.

## Task 3: Sandboxing Spike and Implementation

Resolves the highest-risk architectural unknown before discovery and CLI are designed in detail.

Files: `plugins/runner.py`; sandboxing module and tests in `plugins/`.

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

**Final verification:** Run `pytest tests/plugins/ tests/cli/test_plugins.py -v && python scripts/smoke_plugin_pipeline.py fixtures/plugins/noop_plugin.py` -> PASS and prints `sample plugin executed`.

## Execution Log

*(append entries here as each task completes - format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*

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
