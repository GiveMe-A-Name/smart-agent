# Execution Handoff Reference

## Execution Log

Maintain an `## Execution Log` at the bottom of the plan document during execution. **Never edit the plan tasks section during execution** — it stays static as source of truth.

Each log entry answers: **what was done and why** — not just status. A future agent reading the log should understand the intent behind each change, not just that it happened.

- **Every task**: record what changed and the reasoning behind key decisions — especially for changes that other tasks will also touch. Calibrate detail to how non-obvious the decision is; obvious steps can be brief.
- **Failed tasks**: also record the specific failing test + error output, and why the fix works — enough context to reconstruct a targeted repair without re-investigating.
- **Plan revision triggers**: which assumption broke, exactly how future tasks were updated, and why.

## Plan Revision

If a plan revision trigger fires: update not-yet-started tasks in the plan section and record the reason in the log. Do not rewrite completed or in-progress tasks.
