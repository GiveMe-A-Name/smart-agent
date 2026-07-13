---
name: writing-plans
description: "Create implementation plans that turn a settled target into scoped, verifiable tasks. TRIGGER when: a user asks for a plan, or implementation work has a confirmed goal but affected files, sequencing, risks, or verification are not mapped."
---

# Writing Plans

Turn a confirmed implementation target into a right-sized execution plan.

## Principles

- **Assessment before decomposition.** Do not write tasks until you can state: *"This is a [size] [nature] task. The relevant code/evidence is [X]. The main constraint is [Y]. Done means [observable behavior, contract, or state]. I can prove it with [exact verification command -> expected output]."* [because a plan written before this point can look complete while targeting the wrong files or scope]
- **Right-size the artifact to the uncertainty.** Tiny work gets a prose note; small work gets a short ordered list; medium/large work gets a persistent plan with risks, exclusions, stop signals, and handoff detail [because plan structure has cost, and its value comes from reducing ambiguity the executor would otherwise face]
- **Tasks must deliver behavior or resolve a named uncertainty.** A task that only creates a layer, file category, or scaffolding with no independent verification is a horizontal slice [because integration risk is delayed until the end, where it is most expensive to recover]
- **Plan the contract, not the implementation recipe.** Specify system promises, inputs, outputs, errors, ownership, and verification; leave library choice, local code shape, and small refactors to execution unless they are already decided constraints [because over-specified plans block execution-time judgment when code evidence changes]
- **Make design intent and solution shape visible before decomposition.** If multiple plausible designs could satisfy the same user direction, explain why this approach was chosen and what responsibility shape the code or system should have after the change [because humans need to approve the agent's interpretation of the problem and design direction, not just the final behavior]
- **Different decisions need different reading surfaces.** Put outcome approval in a concise Decision Brief, design approval in a separate Design Review only when a human choice can materially change the solution, and files, commands, and task order in the Execution Plan [because mixing product alignment, architecture review, and cold-start instructions forces every reader through detail unrelated to their decision]
- **Each fact has one canonical home.** Outcome and scope belong in the Decision Brief; architecture and alternatives belong in Design Review; files, commands, and task order belong in the Execution Plan [because repeating the same rationale or sequence across surfaces increases reading cost and lets copies drift]
- **Design detail is conditional on approval risk, not plan size.** Include Design Review when choosing differently would materially change a public contract, safety or data property, ownership boundary, runtime boundary, migration, rollback, or long-term dependency shape [because crossing layers or editing several modules does not by itself create a human design decision]
- **Detail decays with distance.** Fully detail only tasks whose inputs are known now. Later tasks that depend on findings from earlier tasks stay goal-level with revision triggers [because detailed future steps built on unknown outcomes turn assumptions into instructions]

## Situation Assessment Gate

Before writing the plan, record or be able to state these from observable evidence:

- **Intent:** the user-visible or domain-level outcome, in the user's terms.
- **Size:** Tiny / Small / Medium / Large, justified by affected functions, files, modules, interfaces, services, or domains.
- **Nature:** bug fix, feature, refactor, migration, or architecture change. Bug fixes require a named root cause. Refactors require a caller/dependency map. Features require at least one existing pattern read or an explicit note that none exists.
- **Current state:** files or artifacts read, relevant behavior observed, existing patterns to reuse, and constraints that cannot move.
- **Done state:** the observable behavior, contract, or system state that must hold after execution; do not put commands here.
- **Final verification:** exact command(s), expected output, and any necessary manual observation that prove the done state. Determine these before decomposition, but record them only once in the plan's Final Verification.

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

After an optional document title, all plans use Decision Brief as the first substantive section for human outcome approval. Add a separate Design Review only when the Design Review Gate is triggered. The Execution Plan follows both approval surfaces and contains the evidence and instructions needed for cold execution.

Tiny/Small plan body:
- Decision Brief with Intent, Outcome, and Scope; add Visible Changes, Approval Needed, or Primary Risk only when their observable triggers apply
- assessment sentence: size, nature, relevant code/evidence, main constraint, and observable done state
- files or areas involved
- ordered steps or prose note at the depth required by size
- Final Verification with exact command(s) and expected output

Medium/Large plan body:
- Decision Brief with Intent, Outcome, and Scope; conditional Visible Changes, Approval Needed, and Primary Risk
- Design Review before technical assessment when the Design Review Gate is triggered
- technical assessment: size, nature, current state, observable done state, decomposition strategy
- explicit exclusions
- execution risks and plan-internal responses for the riskiest assumptions: a risk names an uncertainty and consequence; a contingency names an alternative path that preserves the approved Decision Brief and Design Review; a revision trigger permits refinement of unapproved task detail only
- stop signals: concrete conditions whose response would change the approved outcome, scope, contract, rationale, or solution shape and therefore require pausing for human decision
- file map: exact existing files for planned modifications; directories/modules for new files when names are not yet certain
- ordered tasks with purpose line, dependencies, concrete changes, verification, and commit point
- Final Verification with the complete final command set and expected output
- `## Execution Log` placeholder described in Execution Handoff

Read `references/planning-methodology.md` only when task boundaries, dependency order, contingencies, or stop signals need more guidance than the constraints above; do not load it based on plan size alone. Read `references/estimation.md` only when the user asks for effort, duration, staffing, or scheduling estimates, or repository convention requires them.

Classify each risk response by whether it stays inside the approved surface. Keep plan-internal responses with execution risks; put approval-changing conditions only in Stop Signals. Do not copy the same condition into both sections. When a Primary Risk needs an operational stop condition, reference it briefly and state only the observable trigger rather than repeating its rationale.

## Design Review Gate

Add `## Design Review` only when repository evidence or the user's request shows at least one approval-relevant consequence:

- the plan changes a public API, persisted schema, event contract, compatibility promise, or externally observable failure policy
- the plan changes authorization, security, data integrity, recovery, migration, or rollback strategy
- the plan adds or changes a service, process, persistence, trust, or external-dependency boundary
- two or more repository-supported designs assign responsibility or dependency direction differently, and the choice materially changes maintenance or extension cost
- the user explicitly asks to approve the technical design or states an architecture constraint that the plan must preserve

Do not trigger Design Review from file count, Medium/Large size, ordinary cross-layer plumbing, or nontrivial branching alone. If the repository exposes one established pattern and the plan reuses it without changing a public contract or risk boundary, keep the design constraint in the Execution Plan.

Use only the forms needed for the approval decision: a responsibility or dependency diagram for ownership, a behavior matrix or pseudocode for branching, a contract diff for API/schema changes, or migration stages for rollout. A Design Review must state the proposed shape, the material alternative or constraint, and why the choice fits the evidence; omit architecture diagrams, data shapes, pseudocode, and other forms that do not change the approval decision.

Do not include full production code, line-level edits, file maps, task sequencing, or speculative abstractions. Put unresolved approval-relevant choices in Approval Needed. Put bounded execution uncertainty in an uncertainty-reduction task, revision trigger, or stop signal rather than inventing a design.

## Task Design Constraints

- Each task opens with one human-language purpose sentence that explains why the task exists at this point in the sequence.
- Each task is a vertical slice or a named uncertainty-reduction task. It must leave the codebase in a working state.
- Existing file paths are exact. New files in medium/large plans name the module/directory and responsibility; exact filenames may remain execution-time judgment unless another task depends on them.
- Planned changes are concrete enough that a cold-start executor can act, but they do not prescribe local implementation details unless those details are part of the approved contract.
- If a Design Review exists, every affected task must preserve its approved boundary, contract, and decision semantics. Use a revision trigger only to refine task detail inside that surface; stop for re-approval before choosing a different shape.
- Verification is executable: command plus expected output. Prose such as "run tests" is not a verification step. Final Verification is the only canonical home for the plan's complete final command set; task-local commands may prove intermediate slices but must not be copied into the assessment.
- Cross-boundary work names interfaces, inputs, outputs, error behavior, owner, and handoff point before parallel or dependent tasks begin.
- The first task passes the cold-start test: read by itself, it names the files or modules to open and the first behavior-changing or uncertainty-resolving action.
- For large plans, no more than the first 2-3 tasks get step-by-step checklists unless later task inputs are already known from evidence. Far tasks whose steps depend on earlier outcomes stay goal-level.

## Decision Brief Constraints

- The first substantive section after an optional document title is `## Decision Brief`, containing **Intent**, **Outcome**, and **Scope** in that order.
- Intent states the problem or value and desired direction without restating Outcome. Outcome is one concise sentence describing what will be observably true after execution. Scope names affected product/domain areas and any adjacent non-area whose preservation is approval-critical.
- Add **Visible Changes** only when evidence shows multiple behavior outcomes, a public/developer contract change, compatibility expectation, migration, restored behavior, or multiple valid interpretations. Use the shortest complete form: scenario bullets, Before/After pairs, a contract diff, a behavior matrix, or invariants.
- Add **Approval Needed** only when a reasonable human could choose differently and that choice changes outcome, compatibility, ownership, dependency, rollout, risk, or scope. Omit the field when no such choice exists.
- Add **Primary Risk** only when one uncertainty could force a different approved outcome, scope, or solution shape. Ordinary implementation difficulty belongs in the technical risks section.
- Do not include Size, task summaries, files, commands, internal APIs, implementation-only names, or task ordering in the Decision Brief.
- If two Decision Brief fields paraphrase the same fact, merge the fact into Intent or Outcome and delete the repetition. Do not summarize task purpose or order in the Decision Brief; those have one canonical home in the Execution Plan.

Read `references/decision-brief.md` when a visible behavior delta needs a compact representation, when conditional-field boundaries are unclear, or when revising the approval surface for duplication. Do not read it for execution-only changes.

## Design Review Constraints

- Design Review appears after the Decision Brief and before Situation Assessment only when the Design Review Gate has observable evidence.
- Include only representations that let the reviewer decide the triggered issue; do not populate a fixed field list.
- Stable architecture anchors, public/internal contract names, compact data shapes, diagrams, and pseudocode are allowed when they expose the approval choice. Exact files, task steps, and private helper details belong in the Execution Plan.
- If implementation evidence invalidates an approved design decision, stop and request re-approval rather than silently changing the frozen approval surface.

Read `references/design-review.md` only after the Design Review Gate triggers and the minimal representation is unclear. It contains omission boundaries and decision-specific examples; do not load it merely because a plan is Medium or Large.

---

## Independent Review

Use independent review only when the plan has evidence of coordination risk. The review exists to catch gaps the planner is least likely to notice: unverified assumptions on the critical path, cross-boundary contract mismatches, and plans that look complete but cannot be executed cold.

**Large — review required.** Large plans affect multiple components or architectural boundaries, so a single planner is likely to overweight the path they just designed and miss downstream coupling.

**Medium — review required by default.** Skip independent review only when all evidence below is present:
- the plan affects one domain and does not introduce or change a public contract, service, process, persistence, trust, external-dependency, or cross-team boundary
- no unresolved risk can change the approved outcome, scope, or design
- fewer than 2 contingencies are needed
- the assessment names an existing codebase pattern already verified in repository evidence

If any skip condition is false or cannot be established from the plan and repository evidence, review is required [because an incomplete skip case is evidence that the planner's assumptions or coordination surface need a fresh check].

**Tiny/Small — review skipped.** The review overhead exceeds value when the plan changes one known behavior path, uses exact files, and has a single verification command.

When review is required, launch a fresh sub-agent reviewer when the environment supports sub-agents. Give it a bounded review packet, not the whole planning conversation: the plan document, the original user-visible intent, non-negotiable user constraints, the frozen Decision Brief and Design Review if one exists, repository instructions or architecture docs named by the plan, and the exact files the plan cites as evidence or modification targets. Ask the reviewer to follow `plan-document-reviewer-prompt.md` and to flag missing context instead of inferring it [because independence catches planner rationalization, while a bounded packet prevents a context-poor reviewer from inventing priorities outside the user's intent].

The planner remains responsible for adjudication. For every Critical or Important reviewer issue, classify it before editing the plan:
- **Accept:** the issue cites plan text, user intent, repository evidence, or a missing context/file condition that can break execution; update the plan.
- **Surface:** the issue depends on a human trade-off or missing user decision; add it to Approval Needed, stop signals, or the handoff note.
- **Reject:** the issue is a preference, asks for scope outside the approved intent, or cannot identify an execution failure mechanism from the review packet; do not change the plan for it.

Do not treat the sub-agent verdict as authoritative. The blocking threshold depends on the adjudicated issues, not raw reviewer output: Large fixes accepted Critical and Important issues before handoff; Medium fixes accepted Critical issues before handoff and surfaces accepted Important issues to the human reviewer. If sub-agents are unavailable, run the same review prompt in a separate pass and note that the review was not independently delegated.

## Approval Contract

Approved Decision Brief and Design Review sections are read-only; any semantic change requires re-approval. See `references/execution-handoff.md` for superseding approvals and plan-revision mechanics.

## Execution Handoff

Save Medium/Large plans to `docs/plans/YYYY-MM-DD-<name>.md`, then offer to execute. Save Tiny/Small plans only when the user asked for a persistent plan or the repo convention requires it. The plan settles scope and targets; execution still requires judgment about how each change lands.

Include an `## Execution Log` section at the bottom of saved plans with exactly this placeholder line:

```
*(append entries here as each task completes - format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*
```

For log entry format, read-only constraints, failed-task recording, and revision mechanics, see `references/execution-handoff.md` before executing or revising a saved plan.

## Design Review Examples

Use the case examples below as behavioral calibration, not as field inventories. Each positive example uses the smallest approval surface sufficient for its evidence; omission is intentional and must be copied as carefully as inclusion.

## Example Case References

Read only the example case that matches the planning problem; do not read every example by default [because examples calibrate judgment, and loading unrelated cases can make the plan imitate the wrong size or failure mode].

| Case | Read when | Reference point |
|------|-----------|-----------------|
| `examples/tiny-bug-fix.md` | Planning a one-path bug fix with a known root cause. | Minimum viable plan: human-readable intent plus one executable note. |
| `examples/small-internal-refactor.md` | Planning a multi-file internal refactor that reuses one established pattern without changing contracts or boundaries. | Crossing files does not trigger Design Review; preserve behavior with a short plan. |
| `examples/medium-webhook-notifications.md` | Planning a medium feature that crosses a few modules but can be proven by a walking skeleton. | Human/agent split, vertical slicing, risk-first ordering, exclusions, stop signals, and executable tasks. |
| `examples/medium-api-contract-change.md` | Planning an additive public contract change with mixed-version rollout. | Contract-focused Design Review using a diff and compatibility semantics rather than a full architecture template. |
| `examples/large-plugin-system.md` | Planning a large architecture change with contracts, dependencies, and later tasks that depend on early findings. | Contract-first planning, dependency ordering, progressive refinement, and plan revision after early findings. |
| `examples/completion-faking-anti-pattern.md` | Checking whether a plan has section-shaped tasks but cannot be executed cold. | Why structure without exact first actions, commands, and referents fakes completeness. |
| `examples/horizontal-slicing-anti-pattern.md` | Checking whether tasks are grouped by layers instead of working behavior. | Why layer-by-layer tasking delays learning and verification. |
