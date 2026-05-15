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
- executing an already-agreed plan where scope, targets, and sequencing are settled [because writing a second plan when one already governs execution creates ambiguity about which document is authoritative]
- still in open-ended exploration, root-cause investigation, or brainstorming with no settled implementation target [because planning without a confirmed target produces a structurally complete document for the wrong problem]

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

You cannot start writing until you can state: *"This is a [size] [nature] task. The relevant code is [X]. The constraint is [Y]. Done means: [exact verification command → expected output]."* [A plan written before reaching this state looks structurally complete while targeting the wrong files or solving the wrong scope — the structure becomes false confidence, not real guidance.]

If you can't name the files involved or state how to verify completion, you haven't assessed enough.

**"Restore old behavior / keep original logic" → ambiguous by construction.** Check whether reachable capability paths exist in the current codebase that weren't present at the referenced baseline. If they do, name the interpretations explicitly and get the user's choice before writing any plan — a mechanical revert that silently closes a reachable path is a regression, not a restoration.

**"Historical behavior + new capability" conflict → behavior matrix required.** Output input conditions × old behavior × current behavior × candidate target behavior. If any cell is a "?" — a behavior the user hasn't confirmed — resolve those cells with the user before planning.

## Completion Criteria

- [ ] The Human Review Section opens with a standalone **Intent** statement — one sentence stating what the user wants to achieve in their terms. This appears before Layer 1 and before all other fields.
- [ ] The plan starts with a Human Review Section (Layer 1 for all sizes; Layer 2 + Task Overview + Key Decisions also for medium/large).
- [ ] **Human Review Section language check:** It contains zero file paths, line numbers, function names, method names, implementation-only class/module names, or non-user-visible CLI flags. Each bullet names a user-visible or domain-level outcome. If a sentence needs codebase knowledge to understand, rewrite it by replacing implementation detail with the user problem, system behavior, or product/domain concept it represents.
- [ ] **Task Overview present (medium/large):** Each task has one plain-English sentence in the Human Review Section explaining what it achieves and why it comes at this point in the sequence.
- [ ] **Key Decisions present (medium/large):** Every design choice requiring human ratification is listed with alternatives considered and reasoning.
- [ ] The plan states task size, nature, current state, and decomposition strategy.
- [ ] Each task in the execution section opens with one sentence in human terms explaining *why this task exists* — what problem it solves or what risk it resolves.
- [ ] Each task is a vertical slice: delivers verifiable value, leaves the codebase working, ordered risk-first.
- [ ] For existing files being modified: paths are exact. For new files in medium/large work: module and direction are clear, exact names are execution-time judgment. Planned changes are concrete, not vague.
- [ ] For medium/large: contingencies for the riskiest assumptions and plan revision triggers identified.
- [ ] For cross-boundary work: interfaces and handoffs are explicit.
- [ ] **Execution cold-start test:** Read the first task in isolation, without any conversation context. The task must name the specific existing files to open or the module/directory where new files belong, and the first behavior-changing action. If the first task requires inferring context from a prior message, revise it.
- [ ] For medium/large: at least one explicit exclusion stated — what is deliberately out of scope.
- [ ] For medium/large: stop signals listed in the plan document — conditions under which the agent must pause and ask before continuing.
- [ ] For medium/large with multiple competing goals: conflict priority stated.
- [ ] **Progressive refinement (large only):** Tasks whose specific steps depend on outcomes from earlier tasks not yet done are written at goal level — one short paragraph stating what the task achieves and what it depends on. They do NOT have step-by-step checklists. Typically only the first 2–3 tasks are fully detailed; if more than 3 tasks have detailed checklists, verify whether the far tasks' steps could change once early tasks complete — if yes, revise them to goal-level [because planning steps in detail before you know the inputs is planning for a future that does not exist yet].

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is artifact-type: completion is verified by self-checking the plan document against the Completion Criteria checklist above. No separate user confirmation is required as a completion step — the Human Review Section embedded in the plan document is the mechanism for user approval of scope and direction.

## Anti-Rationalization Check

This section exists because model introspection is unreliable when the work looks finished — a plan that satisfies all checklist items can still be a document that fails at execution time.

Pause before exiting. Do not treat this section as another checklist to clear.

Did I ignore failure signals (over-planning, under-planning, horizontal slicing) because the plan already looks orderly?

Am I exiting because the plan is genuinely complete, or because the outline now looks structured enough?

Completion-faking signals specific to plan writing — stop if any apply:
- Task why-sentences restate the task name rather than explain why the task exists at this point in the sequence
- Verification steps are written as prose ("run the tests") rather than executable commands with expected output — prose is not a verification step
- Reading the first task requires the planning conversation for context — the plan is not self-contained
- The Human Review Section contains file paths, line numbers, function names, method names, implementation-only class/module names, or non-user-visible CLI flags
- A Human Review Section sentence explains implementation mechanics without naming the user problem, system behavior, or product/domain concept it represents
- For large plans: read the last task in isolation — if its detailed steps depend on specific outcomes from earlier tasks not yet done, progressive refinement was not applied; convert tasks whose content depends on uncertain earlier outcomes to goal-level descriptions
- For large plans: count tasks with step-by-step checklists — if more than 3 have them, verify whether the far tasks' steps could change once early tasks complete; if yes, revise to goal-level

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
- **Contract, not implementation** — specify what the system promises (endpoint, inputs, outputs, error conditions), not how it delivers (which library, which algorithm, which code structure); if a step reads like pseudocode or a code walkthrough, it belongs in execution
- **Task purpose line required** — every task must open with one sentence explaining why it exists in human terms (e.g. *"Proves the approach works before investing in config and retry logic"*), not a restatement of what the checklist does
- **Explicit exclusions** — for medium/large: state what the plan does NOT cover; without this, an agent will extrapolate scope at execution time
- **Conflict priority** — for medium/large with competing goals: name which wins when they conflict (e.g., "correctness over performance; security before schedule")
- **Stop signals** — for medium/large: name conditions that require pausing and asking before continuing; see `references/planning-methodology.md`
- **Uncertainty surfaced, not buried** — if the user expressed doubt about a premise ("I think X works this way", "not sure if", "maybe"), treat it as unverified; name it as an explicit assumption and add a verification step before any task that depends on it [because an unverified assumption embedded in a plan propagates into every dependent task — if the premise is wrong, all downstream work is built on a false foundation]

## Failure Signals

**Over-planning** — stop if:
- Writing TDD ceremony for a one-function change
- Plan document heavier than the work itself
- Running a review loop on a tiny/small task
- Planning tasks 6+ in full detail when tasks 1-3 haven't been done yet [because early tasks will change the shape of later tasks — detailed planning beyond the next 2-3 tasks is planning for a future that does not exist yet]
- Specifying implementation details (library choice, algorithm, code structure) rather than the behavioral contract
- A suggestion the user hedged ("probably", "maybe", "idk", "or something", "should we also?") became a committed task — hedged suggestions require an explicit decision (ask, exclude with a note, or flag as pending); do not silently absorb them as confirmed scope

**Under-planning** — stop if:
- Starting medium/large work without a file map
- Verification written as prose ("run the tests") rather than executable command with expected output [because prose verification cannot be mechanically confirmed — "done" becomes unverifiable and the plan cannot serve as a cold-start execution document]
- Planning a bug fix without knowing the root cause
- No dependency analysis for work crossing multiple modules
- No contingency for the riskiest assumption in a medium/large plan [because the riskiest assumption is where the plan is most likely to break — skipping it produces a plan that is only correct if everything goes right]
- No plan revision triggers — plan assumes everything goes as expected
- Tasks reference context from the planning conversation rather than stating it directly in the document
- No explicit exclusions for medium/large work where an agent could plausibly extend scope
- No stop signals — agent has no defined conditions to pause and ask
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
- The plan states as confirmed something the user expressed uncertainty about ("I think", "not sure", "maybe"), with no verification step [because an unverified assumption embedded in a plan propagates into every dependent task — if wrong, all downstream work is built on a false premise]

---

## Independent Review

Use independent review only when the plan has evidence of coordination risk. The review exists to catch gaps the planner is least likely to notice: unverified assumptions on the critical path, cross-boundary contract mismatches, and plans that look complete but cannot be executed cold.

**Large — review required.** Large plans affect multiple components or architectural boundaries, so a single planner is likely to overweight the path they just designed and miss downstream coupling.

**Medium — review required if any evidence below is present:**
- Any primary risk is rated High [because a High-rated risk marks an unverified assumption on the critical path]
- The plan contains ≥2 contingencies [because multiple contingencies mean the planner already found multiple plausible failure paths]
- The plan spans ≥3 modules, services, teams, or system boundaries [because cross-boundary count is observable evidence that interface mismatch can break the plan]

**Medium — review skipped only when all evidence below is present:**
- exactly one domain or module boundary is affected
- no primary risk is rated High
- fewer than 2 contingencies are listed
- the plan names an existing codebase pattern already verified in the assessment

**Tiny/Small — review skipped.** The review overhead exceeds value when the plan changes one known behavior path, uses exact files, and has a single verification command.

When review is required, use the independent-review guidance in `plan-document-reviewer-prompt.md`. The blocking threshold depends on plan size: Large fixes Critical and Important issues before handoff; Medium fixes Critical issues before handoff and surfaces Important issues to the human reviewer.

## Human Review Section

Every plan document starts with a **Human Review Section** — a self-contained block written for human approval, not agent execution. Place it at the very top, before the assessment and technical tasks. This section is a contract: once approved, it is frozen and must never be modified for any reason.

The Human Review Section must open with a one-sentence **Intent** statement before all other fields. This is the alignment checkpoint: the user reads it first, confirms or corrects direction, then decides whether to review the rest.

For fields, scaling rules, freeze semantics, and Before/After examples — see `references/human-readable-summary.md`.

## Execution Handoff

Save to `docs/plans/YYYY-MM-DD-<name>.md`, then offer to execute. The plan settles scope and targets; execution still requires judgment about how each change lands.

Include an `## Execution Log` section at the bottom of the plan document — section created at plan time with exactly this placeholder line (do not leave it blank):

```
*(append entries here as each task completes — format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*
```

For log entry format, read-only constraints, failed-task recording, and revision mechanics — see `references/execution-handoff.md`.

---

## Planning Methodology

For detailed planning guidance — how to assess task size, decomposition strategies, estimation, and contingency planning — see `references/planning-methodology.md`.

For Tiny/Small/Medium/Large plan examples — see `examples.md`.
