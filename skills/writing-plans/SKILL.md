---
name: writing-plans
description: "Turn a settled goal or issue into a right-sized, verifiable execution plan. Use when a user asks for a plan, or when implementation work has a confirmed target but affected files, sequencing, risks, or verification are not yet mapped."
---

# Writing Plans

Turn a confirmed target into a right-sized execution plan that guides implementation and catches wrong assumptions early.

## Assess Before Decomposing

Treat issue descriptions, guessed root causes, and file hints as hypotheses until verified against actual code. Before listing tasks, establish from repository evidence:

- **Outcome and constraints:** the observable behavior, contract, or state that must hold when done, plus constraints that cannot move.
- **Current state:** real files read and relevant code observed—a confirmed root cause for bug fixes, callers and dependencies for refactors, or an existing pattern (or explicit note that none exists) for features.
- **Final verification:** the exact command(s), expected output, and any required manual check that prove the done state.

**Baseline-restoration check:** When asked to restore old behavior or keep original logic, inspect whether current code has reachable capabilities absent from the referenced baseline. Do not plan a mechanical revert that silently removes a live capability without confirming which behavior should win.

## Right-Size and Place the Plan

Match plan weight to uncertainty rather than producing a formal document by default:

- **Small / straightforward work** (localized scope, existing patterns apply, no major unknowns): keep an ephemeral inline plan in context—target files, ordered steps, and verification command—and proceed directly.
- **Medium / large work** (cross-module impact, new contracts, or material unknowns): write the plan to a temporary file outside git tracking (such as `/tmp/` or the agent's scratch directory) rather than committing it into the repository, unless the user or repository convention asks for a tracked plan. Update not-yet-started tasks in place as earlier tasks reveal new facts.

## Decompose for Fast Feedback

- **Vertical slices and bounded spikes:** Every task must deliver verifiable behavior or resolve a named uncertainty, leaving the codebase in a working state. Do not group tasks by layers (models, infrastructure, wiring, then tests) that delay end-to-end verification until the end.
- **Front-load risk and contracts:** Put the riskiest unverified assumption or a minimal end-to-end walking skeleton first. For cross-module changes, settle interfaces (inputs, outputs, error behavior, ownership) before building dependent sides.
- **Plan contracts, not recipes:** Specify behavior boundaries, concrete target files (or module/directory for new files whose names are not yet fixed), and executable verification (`command -> expected output`; never prose such as "run tests"). Leave local code shape and implementation idioms to execution unless they are hard constraints.
- **Detail decays with distance:** Fully detail only the first 2–3 tasks whose inputs are verified now. Keep later tasks goal-level with a revision trigger after the task that resolves their unknowns.

## Autonomous Guardrails

- **Explicit exclusions and tiebreakers:** For medium/large work, state 1–3 plausible adjacent things deliberately left out of scope to prevent drift and over-engineering. When competing goals exist (such as correctness vs. performance), state which wins.
- **Escalate instead of guessing:** Adapt task order, internal structure, and local implementation choices autonomously as code evidence emerges. Stop and escalate (for example, comment on the issue or ask the user) only when a premise breaks: intent or root cause cannot be established, the request contradicts repository reality, or the only viable path breaks an unmentioned public contract, requires a risky migration, or introduces an unapproved external dependency or credential.
- **Honest final verification:** Before finishing, reconcile actual executed command results against the planned final verification. Report passed, failed, and unrun checks explicitly—a narrowed check or pre-existing failure is not a full pass.

Read `examples/progressive-execution-plan.md` when planning medium or large work with early unknowns and you want a concrete reference for compact structure, vertical slicing, and progressive refinement.
