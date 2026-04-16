---
name: writing-plans
description: "Plan before implementing when the path is unclear. TRIGGER when: you cannot immediately state the files involved, what changes, and how to verify it. DO NOT TRIGGER when: executing an existing plan, or the change is trivially scoped to a single known file."
---

# Writing Plans

Plan before writing code when the path is unclear. A plan that doesn't match task size is waste — tiny work gets a prose note, large work gets full decomposition.

## Trigger Logic

**Invocation default**: Skipping a plan converts unknown unknowns into mid-implementation surprises. Even a Tiny plan forces the situation assessment that prevents wasted work. Invoke before writing code unless executing a plan that already exists and has no remaining decisions.

Use this skill when:
- you cannot immediately state the files involved, what specifically changes, and how to verify it
- the work involves multiple steps, files, or domains where sequencing or task boundaries affect the outcome
- the work is a refactor, migration, or architecture change where mapping what touches what matters before moving anything

Do not use this skill when:
- executing an already-agreed plan where scope, targets, and sequencing are settled
- still in open-ended exploration, root-cause investigation, or brainstorming with no settled implementation target

If prerequisite understanding is still missing — codebase evidence thin, or intent unclear — build that first.

## Capability Boundary

This skill owns the full planning cycle: from reading a spec to a plan ready for implementation.

Does NOT own:
- Execution of the plan
- Spec creation or open-ended brainstorming
- Root cause investigation
- Engineering judgment about HOW each change should land — that belongs to execution

Does define the plan document structure; execution log maintenance rules live in `references/execution-handoff.md`.

## Invariants

```
NO PLAN WITHOUT SITUATION ASSESSMENT FIRST
```

You cannot start writing until you can state: *"This is a [size] [nature] task. The relevant code is [X]. The constraint is [Y]. Done means: [exact verification command → expected output]."*

If you can't name the files involved or state how to verify completion, you haven't assessed enough.

**"Restore old behavior / keep original logic" → ambiguous by construction.** Check whether reachable capability paths exist in the current codebase that weren't present at the referenced baseline. If they do, name the interpretations explicitly and get the user's choice before writing any plan — a mechanical revert that silently closes a reachable path is a regression, not a restoration.

**"Historical behavior + new capability" conflict → behavior matrix required.** Output input conditions × old behavior × current behavior × candidate target behavior. If any cell is a "?" — a behavior the user hasn't confirmed — resolve those cells with the user before planning.

## Completion Criteria

- [ ] The plan starts with a human-readable summary (Layer 1 for all sizes; Layer 2 also for medium/large).
- [ ] **Human readability litmus test:** Read Layer 1 aloud. If a product manager who doesn't know this codebase couldn't understand it, rewrite. Layer 1 must use domain language — no file paths, no method names, no code references.
- [ ] The plan states task size, nature, current state, and decomposition strategy.
- [ ] Each task opens with one sentence in human terms explaining *why this task exists* — what problem it solves or what risk it resolves, not what it does technically.
- [ ] Each task is a vertical slice: delivers verifiable value, leaves the codebase working, ordered risk-first.
- [ ] For existing files being modified: paths are exact. For new files in medium/large work: module and direction are clear, exact names are execution-time judgment. Planned changes are concrete, not vague.
- [ ] For medium/large: contingencies for the riskiest assumptions and plan revision triggers identified.
- [ ] For cross-boundary work: interfaces and handoffs are explicit.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting. Do not treat this section as another checklist to clear.

Did I ignore failure signals (over-planning, under-planning, horizontal slicing) because the plan already looks orderly?

Am I exiting because the plan is genuinely complete, or because the outline now looks structured enough?

---

## Task Sizing

Calibrate plan depth to task size:

| Size | Scope | Plan depth |
|------|-------|------------|
| **Tiny** | One function/method, no new interfaces, no new test design | Prose note: file, change, verification command |
| **Small** | Crosses function boundaries, one domain, existing patterns apply | Short list: files, order, checks |
| **Medium** | New interface or concept, multiple domains, new test design | Ordered tasks with risks, commit points, contingencies |
| **Large** | Architecture decisions, multi-component impact | Full decomposition; near tasks detailed, far tasks goal-level |

## Quality Principles

Regardless of size, a usable plan is concrete:
- **File paths match certainty** — exact for existing files being modified; directional for new files in medium/large work ("discovery + validation modules in `plugins/`", not just "somewhere in src/")
- **Concrete changes** — not "add error handling", but what the code should actually do
- **Specific verification** — executable command with expected output; "run the tests" is not a verification step, `pytest tests/foo.py::test_bar -v → PASS` is
- **One thing per step** — if a step has "and", it's two steps
- **Vertical slice per task** — delivers verifiable value, not just a layer
- **Working state after each task** — commit point rule applies at every level
- **Targets, not methods** — if a step reads like pseudocode or a code walkthrough, it belongs in execution, not the plan
- **Task purpose line required** — every task must open with one sentence explaining why it exists in human terms (e.g. *"Proves the approach works before investing in config and retry logic"*), not a restatement of what the checklist does

## Failure Signals

**Over-planning** — stop if:
- Writing TDD ceremony for a one-function change
- Plan document heavier than the work itself
- Running a review loop on a tiny/small task
- Planning tasks 6+ in full detail when tasks 1-3 haven't been done yet
- Describing HOW to implement rather than WHAT changes and WHERE

**Under-planning** — stop if:
- Starting medium/large work without a file map
- Verification written as prose ("run the tests") rather than executable command with expected output
- Planning a bug fix without knowing the root cause
- No dependency analysis for work crossing multiple modules
- No contingency for the riskiest assumption in a medium/large plan
- No plan revision triggers — plan assumes everything goes as expected
- Cross-boundary work with no agreed interfaces or handoff points

**Horizontal slicing** — stop if:
- All tasks in one layer ("create all models", "create all endpoints")
- No task delivers end-to-end verifiable behavior
- The first task you can actually test is the last one
- Tasks leave the codebase in a non-working state

**Skipping assessment** — stop if:
- You started writing before reading the relevant code
- You don't know which files are involved
- You haven't stated the decomposition strategy

---

## Review Loop

For medium/large plans only: dispatch a plan-document-reviewer subagent (see `plan-document-reviewer-prompt.md`). Skip for tiny/small — overhead exceeds value.

## Human-Readable Summary

Every plan document starts with a summary written for human reviewers, not agents. Place it at the very top, before the assessment and technical tasks. Tiny/Small plans need only Layer 1 (overview); Medium/Large plans need both layers.

For structure, scaling rules, and update rules — see `references/human-readable-summary.md`.

## Execution Handoff

Save to `docs/superpowers/plans/YYYY-MM-DD-<name>.md`, then offer to execute. The plan settles scope and targets; execution still requires judgment about how each change lands.

Include an `## Execution Log` section at the bottom of the plan document — section created at plan time with exactly this placeholder line (do not leave it blank):

```
*(append entries here as each task completes — format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*
```

Entries are written during execution. Three hard constraints apply:

- **Never edit the plan tasks section during execution** — it stays static as source of truth
- **Each entry must answer what was done and why** — not just status
- **Revision only updates not-yet-started tasks** — completed and in-progress tasks are never rewritten

For log entry format, failed-task recording, and revision mechanics — see `references/execution-handoff.md`.

---

## Planning Methodology

For detailed planning guidance — how to assess task size, decomposition strategies, estimation, and contingency planning — see `references/planning-methodology.md`.

For Tiny/Small/Medium/Large plan examples — see `examples.md`.
