# Execution Handoff Reference

## Read-Only Constraints During Execution

These parts of the plan document have read-only constraints once execution begins:

1. **Decision Brief and any Design Review** — frozen separately at approval. Read them as the outcome and design contracts; never write to them silently. If execution requires a different outcome, scope, approved choice, boundary, contract, dependency direction, failure policy, migration, or rollout, stop and ask the human to approve a superseding section.

2. **Completed and in-progress tasks** — frozen as a historical record. Only not-yet-started tasks may be revised.

## Execution Log

Maintain an `## Execution Log` at the bottom of the plan document during execution. During normal execution, the plan tasks section stays static as the source of truth. Edit only not-yet-started tasks, and only when a named plan revision trigger fires.

Each log entry answers: **what was done and why** — not just status. A future agent reading the log should understand the intent behind each change, not just that it happened.

- **Every task**: record what changed and the reasoning behind key decisions — especially for changes that other tasks will also touch. Calibrate detail to how non-obvious the decision is; obvious steps can be brief.
- **Failed tasks**: also record the specific failing test + error output, and why the fix works — enough context to reconstruct a targeted repair without re-investigating.
- **Plan revision triggers**: which assumption broke, exactly how future tasks were updated, and why.

## Plan Revision

If a plan revision trigger fires without changing an approved outcome or design: update not-yet-started tasks in the plan section and record the reason in the log. Do not rewrite completed or in-progress tasks.

If the revision changes approved content, leave the original Decision Brief or Design Review intact, add a clearly marked superseding approval section, and wait for approval before continuing.

## Execution Log Example

```text
[2026-04-06] Task 1: complete. Defined the plugin contract and sample plugin.
Moved the shared pipeline data shape behind the plugin contract to preserve the
approved one-way dependency and avoid a circular import.

[2026-04-07] Task 3: failed first attempt. The sandbox test showed an import path
could bypass restricted globals. Switched to subprocess isolation; the focused
sandbox suite passed.

Plan revision triggered. Because plugins are now out-of-process, not-yet-started
discovery work was revised to read manifest metadata without importing plugin code.
```
