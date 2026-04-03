---
name: implementation-judgment
description: "Invoke before writing any behavior-affecting code — bug fixes, new features, review responses, and small patches alike. The structural questions this skill addresses (right layer, correct responsibility placement, silently broken contract) appear in all code changes, not only large design decisions. Cost of unnecessary invocation: a few extra reasoning steps. Cost of missing: a patch that weakens structure, moves responsibility to the wrong layer, or locks in a wrong abstraction — often invisibly."
---

# Implementation Judgment

Implement changes with engineering judgment, not just task completion.

## Trigger Logic

**Invocation default**: The trigger state is: any behavior-affecting code is about to be written. This applies to bug fixes, new features, review responses, and small patches equally — the structural questions this skill addresses (wrong layer, wrong responsibility owner, silently broken contract) are not proportional to change size. Do not gate this skill on perceived complexity. The cost of a missed invocation is a patch that weakens structure, introduces a subtle failure mode, or locks in a wrong abstraction — often invisibly.

**Safe to skip**: only when the change has confirmed zero behavioral effect — a rename with no logic change, a file move, a comment fix, pure formatting. These must be observable facts already confirmed from the code, not impressions formed before reading it.

**Knowing where to change is not the same as knowing how.** Prior investigation, planning, or root-cause analysis grounds the decision — it does not substitute for this judgment.

**Prerequisites**: This skill requires code evidence and clarified intent. If code evidence is missing, build that understanding first. If intent is unclear, clarify it first. Both are sequencing dependencies — work to complete before this skill, not reasons to skip it.

## Boundary

This skill owns:
- reasoning from clarified intent and grounded code evidence to responsible implementation
- judging whether candidate abstractions, patterns, and boundaries should be preserved, extended, corrected, or removed
- deciding where complexity should live and who should own each responsibility
- evaluating failure modes, edge cases, and implicit contracts before writing code
- deciding whether a local patch is sufficient, a focused structural improvement is warranted, or an explicit pause is needed
- balancing delivery speed, simplicity, system coherence, and future maintainability
- deciding how accepted review feedback should land without introducing structural damage

This skill does not own:
- product clarification from scratch
- repository analysis as an end in itself
- generic task planning
- unrestricted refactoring beyond what the current change needs
- final completion or release judgment
- evaluating whether review feedback should be accepted

## Invariants

- **Never treat the task description as the full implementation truth** — it names what to change, not whether that is the right layer or abstraction.
- **Never assume existing code shape is arbitrary** without first tracing what it may be protecting — code shaped around implicit contracts, deployment ordering, or compatibility constraints is not arbitrary.
- **Never infer unconfirmed product intent from code shape alone** — structure reflects past decisions, not current requirements.
- **Never let deadline or delivery pressure justify moving responsibility into the wrong layer** — the compounding cost of a misplaced responsibility outlasts the deadline.
- **The minimum viable change is only correct when it does not deepen a structural concern** — a patch that compounds a bad boundary is cumulative damage, not conservatism.
- **Never use theoretical justification as evidence** — "this supports testability," "this could be useful," "this is clean design" are arguments. Evidence is: who calls this, who depends on this, what breaks if this is removed.
- **Never treat duplication removal as inherently correct** — duplication is cheaper than the wrong abstraction.

## Decision Signals

**Abstraction threshold**: Extract when logic appears in 3+ places with identical semantics AND the same rate of change. Keep separate when 2 places exist or the copies are diverging — shared code that changes for different reasons creates a coupling point that makes both cases harder to change.

**Complexity placement**: If callers must know implementation details to use a function correctly, the abstraction boundary is in the wrong place — pull complexity downward into the module.

**Failure path ownership**: Name the recovery owner before adding error handling. The right owner is usually the outermost caller with full context — not the inner function that detected the problem. Scattered partial recovery across layers is worse than explicit propagation with a single owner.

**Contract change detection**: Callers depend on observed behavior, not documented behavior. Any change to default return values, side effects, or error types is a contract change — regardless of what the docs say.

**Dependency direction**: New imports crossing package or module boundaries are structural commitments. Before adding one, confirm the dependency flows from unstable toward stable code — not the reverse.

**State tracing**: Before any refactor that moves code across module boundaries, explicitly enumerate every piece of state the code reads or writes. Missing one creates a consistency window that did not exist before.

**Scope trigger**: If one judgment dimension raises a concern — scope a focused improvement. If multiple dimensions conflict — pause and make the conflict explicit. If none fire — narrow patch. For detailed scope decision support, see `references/risk-triage.md`.

## Completion Criteria

- [ ] The diff is scoped to what the task requires — no files, functions, or abstractions added beyond direct necessity.
- [ ] If the change touches shared state, reads and writes were traced across all affected boundaries.
- [ ] If the change requires migration (schema, API, config), the migration path is safe for production.
- [ ] If invoked after receiving code review, the reviewer's intent was implemented without damaging design integrity.
- [ ] If the change makes an existing capability path unreachable, that impact was surfaced explicitly before implementation.
- [ ] If implementing toward a goal identified by a reviewer, that goal was confirmed by the user rather than only inferred.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Did I add anything justified only by "this supports testability," "this could be useful," or "this is clean design" — without a concrete caller or usage evidence in the existing code?

Am I exiting because the implementation is genuinely sound, or because the patch looks tidy enough?

## Scope Decision

After working through the relevant judgment dimensions:

- **Narrow local patch**: change is well-bounded, existing structure supports it cleanly, no judgment dimension raised a concern.
- **Focused structural improvement**: a purely local patch would deepen a problem identified by one or more judgment dimensions. Scope tightly to that concern — do not broaden into redesign.
- **Explicit pause**: you do not yet understand why the code is shaped the way it is, OR multiple dimensions point in conflicting directions.

For detailed scope decision heuristics, see `references/risk-triage.md`.

## Self-Correction Signals

Stop and revise when:

- you are implementing directly from the request without understanding the surrounding code
- you are treating the first editable location as the right implementation location
- you are using green tests or delivery pressure to justify a patch at a boundary you already suspect is wrong
- you are removing duplication without checking whether the duplicated code is actually diverging
- you are adding parameters or conditionals to an existing abstraction instead of questioning whether the abstraction is still right
- you are scattering error handling across layers without a clear owner
- you are expanding the work into redesign beyond what the change needs
- you are implementing accepted review feedback by copying the reviewer's suggestion literally without applying structural judgment to how it should land
- you are introducing an abstraction without a second concrete caller that demonstrates the need
- you are making assumptions about requirements rather than surfacing ambiguity
- you are moving code across module boundaries without tracing every piece of shared state the code reads or writes
- you are modifying a schema, API response shape, or configuration format without considering the migration path for existing consumers
- you are about to make an existing capability path unreachable without having surfaced this to the user
- you are implementing toward a goal named by a reviewer — not explicitly confirmed by the user
- you are making changes to adjacent code not required by the task rather than noting them for later
- you are using an external API, function signature, or library behavior from memory without verifying against actual source or documentation
- you have worked through multiple judgment dimensions on a small change without reaching a clear decision — stop analysis, pick the simpler option, and write the code

---

For detailed reasoning heuristics on each structural concern, see `references/judgment-dimensions.md`.
For scope decision support, see `references/risk-triage.md`.
For code review response guidance, see `references/review-response-judgment.md`.
For AI-specific failure patterns and explanations, see `references/ai-agent-pitfalls.md`.
