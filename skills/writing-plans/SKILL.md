---
name: writing-plans
description: "Create implementation plans that turn a settled target into scoped, verifiable tasks. TRIGGER when: a user asks for a plan, or implementation work has a confirmed goal but affected files, sequencing, risks, or verification are not mapped."
---

# Writing Plans

Turn a confirmed implementation target into a right-sized execution plan.

## Principles

- **Assessment before decomposition.** Do not write tasks until you can state: *"This is a [size] [nature] task. The relevant code/evidence is [X]. The main constraint is [Y]. Done means [exact verification command -> expected output]."* [because a plan written before this point can look complete while targeting the wrong files or scope]
- **Right-size the artifact to the uncertainty.** Tiny work gets a prose note; small work gets a short ordered list; medium/large work gets a persistent plan with risks, exclusions, stop signals, and handoff detail [because plan structure has cost, and its value comes from reducing ambiguity the executor would otherwise face]
- **Tasks must deliver behavior or resolve a named uncertainty.** A task that only creates a layer, file category, or scaffolding with no independent verification is a horizontal slice [because integration risk is delayed until the end, where it is most expensive to recover]
- **Plan the contract, not the implementation recipe.** Specify system promises, inputs, outputs, errors, ownership, and verification; leave library choice, local code shape, and small refactors to execution unless they are already decided constraints [because over-specified plans block execution-time judgment when code evidence changes]
- **Human approval and agent execution need different surfaces.** The Human Review Section uses user/domain language with no file paths; the execution section names exact existing files, directories for new files, first actions, and commands [because humans approve outcomes and tradeoffs, while agents need cold-start instructions]
- **Detail decays with distance.** Fully detail only tasks whose inputs are known now. Later tasks that depend on findings from earlier tasks stay goal-level with revision triggers [because detailed future steps built on unknown outcomes turn assumptions into instructions]

## Situation Assessment Gate

Before writing the plan, record or be able to state these from observable evidence:

- **Intent:** the user-visible or domain-level outcome, in the user's terms.
- **Size:** Tiny / Small / Medium / Large, justified by affected functions, files, modules, interfaces, services, or domains.
- **Nature:** bug fix, feature, refactor, migration, or architecture change. Bug fixes require a named root cause. Refactors require a caller/dependency map. Features require at least one existing pattern read or an explicit note that none exists.
- **Current state:** files or artifacts read, relevant behavior observed, existing patterns to reuse, and constraints that cannot move.
- **Done state:** exact final verification command(s) and expected output.

Stop before planning if intent, affected area, or final verification is unknown. For bug fixes, also stop if the root cause is unknown. Other uncertainties may become early plan tasks only when they are named, bounded, and have their own verification signal.

Special ambiguity checks:
- **"Restore old behavior" / "keep original logic":** check whether current code has reachable capability paths absent from the referenced baseline. If yes, list the interpretations and ask which target behavior should win before planning [because a mechanical revert can silently remove a real current capability].
- **Historical behavior plus new capability:** create an input conditions x old behavior x current behavior x target behavior matrix. If any target cell is unknown, resolve it before planning.
- **Hedged user suggestions:** if the user says "maybe", "probably", "not sure", "or something", or similar, do not convert the suggestion into committed scope. Ask, exclude with a note, or mark it as a pending decision [because unconfirmed scope becomes invisible work during execution].

## Task Sizing

| Size | Observable scope | Plan depth |
|------|------------------|------------|
| **Tiny** | One function/method, no new interface, no new test design | Prose note: file, change, verification command |
| **Small** | Crosses function boundaries, one domain, existing patterns apply | Short ordered list: files, sequence, checks |
| **Medium** | New interface/concept, multiple domains, or new test design | Persistent plan with ordered tasks, risks, exclusions, contingencies |
| **Large** | Architecture decisions, multi-component impact, or cross-boundary coordination | Full decomposition; near tasks detailed, far tasks goal-level |

## Plan Shape

All plan sizes need a Human Review Section first. See `references/human-readable-summary.md` when writing or revising that section's fields, freeze rules, or examples.

Tiny/Small plan body:
- assessment sentence: size, nature, relevant code/evidence, main constraint, done state
- files or areas involved
- ordered steps or prose note at the depth required by size
- exact verification command(s) with expected output

Medium/Large plan body:
- Human Review Section with Intent, Layer 1, Layer 2, Task Overview, Key Decisions or `None requiring human ratification`, and conflict priority when goals can compete
- technical assessment: size, nature, current state, decomposition strategy
- explicit exclusions
- risks, contingencies, and revision triggers for the riskiest assumptions
- stop signals: concrete conditions that require pausing for human decision
- file map: exact existing files for planned modifications; directories/modules for new files when names are not yet certain
- ordered tasks with purpose line, dependencies, concrete changes, verification, and commit point
- `## Execution Log` placeholder described in Execution Handoff

For detailed decomposition, estimation, contingency, and stop-signal guidance, see `references/planning-methodology.md` when the plan is medium/large or when a small plan has unresolved sequencing risk.

## Task Design Constraints

- Each task opens with one human-language purpose sentence that explains why the task exists at this point in the sequence.
- Each task is a vertical slice or a named uncertainty-reduction task. It must leave the codebase in a working state.
- Existing file paths are exact. New files in medium/large plans name the module/directory and responsibility; exact filenames may remain execution-time judgment unless another task depends on them.
- Planned changes are concrete enough that a cold-start executor can act, but they do not prescribe local implementation details unless those details are part of the approved contract.
- Verification is executable: command plus expected output. Prose such as "run tests" is not a verification step.
- Cross-boundary work names interfaces, inputs, outputs, error behavior, owner, and handoff point before parallel or dependent tasks begin.
- The first task passes the cold-start test: read by itself, it names the files or modules to open and the first behavior-changing or uncertainty-resolving action.
- For large plans, no more than the first 2-3 tasks get step-by-step checklists unless later task inputs are already known from evidence. Far tasks whose steps depend on earlier outcomes stay goal-level.

## Human Review Section Constraints

- The plan begins with a standalone **Intent** sentence before Layer 1 and before every other field.
- Layer 1 appears for all sizes; Layer 2, Task Overview, and Key Decisions appear for medium/large.
- Human Review Section text contains zero file paths, line numbers, function names, method names, implementation-only class/module names, or non-user-visible CLI flags.
- Every Human Review Section bullet names a user-visible, developer-visible, or domain-level outcome. If a sentence requires codebase knowledge to understand, rewrite it using the user problem, system behavior, or product/domain concept.
- For medium/large, each Task Overview item states what the task achieves and why it comes at that point in the order.
- Key Decisions list only decisions a reasonable human might choose differently, with alternatives and reasoning. If no such decision exists in a medium/large plan, write `None requiring human ratification` rather than inventing one.
- If medium/large goals can compete, Conflict Priority appears in the Human Review Section.

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

When review is required, launch a fresh sub-agent reviewer when the environment supports sub-agents. Give it a bounded review packet, not the whole planning conversation: the plan document, the original user-visible intent, non-negotiable user constraints, the frozen Human Review Section if it already exists, repository instructions or architecture docs named by the plan, and the exact files the plan cites as evidence or modification targets. Ask the reviewer to follow `plan-document-reviewer-prompt.md` and to flag missing context instead of inferring it [because independence catches planner rationalization, while a bounded packet prevents a context-poor reviewer from inventing priorities outside the user's intent].

The planner remains responsible for adjudication. For every Critical or Important reviewer issue, classify it before editing the plan:
- **Accept:** the issue cites plan text, user intent, repository evidence, or a missing context/file condition that can break execution; update the plan.
- **Surface:** the issue depends on a human trade-off or missing user decision; add it to Key Decisions, stop signals, or the handoff note.
- **Reject:** the issue is a preference, asks for scope outside the approved intent, or cannot identify an execution failure mechanism from the review packet; do not change the plan for it.

Do not treat the sub-agent verdict as authoritative. The blocking threshold depends on the adjudicated issues, not raw reviewer output: Large fixes accepted Critical and Important issues before handoff; Medium fixes accepted Critical issues before handoff and surfaces accepted Important issues to the human reviewer. If sub-agents are unavailable, run the same review prompt in a separate pass and note that the review was not independently delegated.

## Human Review Section

Every plan document starts with a **Human Review Section** — a self-contained block written for human approval, not agent execution. Place it at the very top, before the assessment and technical tasks. This section is a contract: once approved, it is frozen and must never be modified for any reason.

The Human Review Section must open with a one-sentence **Intent** statement before all other fields. This is the alignment checkpoint: the user reads it first, confirms or corrects direction, then decides whether to review the rest.

For fields, scaling rules, freeze semantics, and Before/After examples — see `references/human-readable-summary.md`.

## Execution Handoff

Save Medium/Large plans to `docs/plans/YYYY-MM-DD-<name>.md`, then offer to execute. Save Tiny/Small plans only when the user asked for a persistent plan or the repo convention requires it. The plan settles scope and targets; execution still requires judgment about how each change lands.

Include an `## Execution Log` section at the bottom of saved plans with exactly this placeholder line:

```
*(append entries here as each task completes - format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*
```

For log entry format, read-only constraints, failed-task recording, and revision mechanics, see `references/execution-handoff.md` before executing or revising a saved plan.

---

## Planning Methodology

For detailed planning guidance, see `references/planning-methodology.md` when task sizing, decomposition, estimation, contingency planning, or stop signals need more detail than the main file provides.

## Example Case References

Read only the example case that matches the planning problem; do not read every example by default [because examples calibrate judgment, and loading unrelated cases can make the plan imitate the wrong size or failure mode].

| Case | Read when | Reference point |
|------|-----------|-----------------|
| `examples/tiny-bug-fix.md` | Planning a one-path bug fix with a known root cause. | Minimum viable plan: human-readable intent plus one executable note. |
| `examples/medium-webhook-notifications.md` | Planning a medium feature that crosses a few modules but can be proven by a walking skeleton. | Human/agent split, vertical slicing, risk-first ordering, exclusions, stop signals, and executable tasks. |
| `examples/large-plugin-system.md` | Planning a large architecture change with contracts, dependencies, and later tasks that depend on early findings. | Contract-first planning, dependency ordering, progressive refinement, and plan revision after early findings. |
| `examples/completion-faking-anti-pattern.md` | Checking whether a plan has section-shaped tasks but cannot be executed cold. | Why structure without exact first actions, commands, and referents fakes completeness. |
| `examples/horizontal-slicing-anti-pattern.md` | Checking whether tasks are grouped by layers instead of working behavior. | Why layer-by-layer tasking delays learning and verification. |
