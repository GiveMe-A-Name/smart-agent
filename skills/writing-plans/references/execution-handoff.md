# Execution Handoff Reference

## Read-Only Constraints During Execution

Two sections of the plan document are frozen once execution begins and must never be modified:

1. **Human Review Section** — frozen at approval. It is a contract between the human and the agent. Read it for context; never write to it. If execution reveals the contract is fundamentally wrong (goal or approach changed), stop and ask the human to re-approve a revised plan — do not silently update the Human Review Section.

2. **Completed and in-progress tasks** — frozen as a historical record. Only not-yet-started tasks may be revised.

## Execution Log

Maintain an `## Execution Log` at the bottom of the plan document during execution. **Never edit the plan tasks section during execution** — it stays static as source of truth.

Each log entry answers: **what was done and why** — not just status. A future agent reading the log should understand the intent behind each change, not just that it happened.

- **Every task**: record what changed and the reasoning behind key decisions — especially for changes that other tasks will also touch. Calibrate detail to how non-obvious the decision is; obvious steps can be brief.
- **Failed tasks**: also record the specific failing test + error output, and why the fix works — enough context to reconstruct a targeted repair without re-investigating.
- **Plan revision triggers**: which assumption broke, exactly how future tasks were updated, and why.

## Plan Revision

If a plan revision trigger fires: update not-yet-started tasks in the plan section and record the reason in the log. Do not rewrite completed or in-progress tasks.
