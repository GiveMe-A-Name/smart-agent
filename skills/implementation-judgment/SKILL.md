---
name: implementation-judgment
description: "Apply engineering judgment before writing behavior-affecting code — right layer, correct responsibility, contract integrity. TRIGGER when: about to write code that changes runtime behavior (bug fixes, features, patches, review-feedback implementation)."
---

# Implementation Judgment

Implement changes with engineering judgment, not just task completion.

## Trigger Logic

**Invocation default**: The trigger state is: any behavior-affecting code is about to be written. This applies to bug fixes, new features, review responses, and small patches equally — the structural questions this skill addresses (wrong layer, wrong responsibility owner, silently broken contract) are not proportional to change size. Do not gate this skill on perceived complexity. The cost of a missed invocation is a patch that weakens structure, introduces a subtle failure mode, or locks in a wrong abstraction — often invisibly.

**Safe to skip**: only when the change has confirmed zero behavioral effect — a rename with no logic change, a file move, a comment fix, pure formatting. These must be observable facts already confirmed from the code, not impressions formed before reading it [because "this looks like a simple rename" is an impression, not evidence — skipping based on impressions converts unknown structural concerns into invisible debt].

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
- **Never produce implementation while a structural conflict is unresolved** — surface the conflict and wait for a resolution decision before writing any code [once code exists, pressure shifts toward making it work rather than questioning the approach; the correct sequence is surface → resolve → implement, not implement → mention].
- **Request framing describes intent, not structural complexity** — "one-liner," "straightforward refactor," and "quick fix" are descriptions of priority, not of structural analysis. When framing suggests simplicity and code evidence reveals complexity, that gap is a risk signal: the requester's model of the change is incomplete [acting on framing rather than evidence is how structural regressions get introduced under authority pressure].

## Decision Signals

**Abstraction threshold**: The primary question is not how many times the code appears, but what would cause it to change — and whether that cause is shared across all copies. Ask: "What event would require me to modify this code?" If every copy has the same answer, extraction reduces drift risk. If any copy has a different answer, extraction creates a coupling point that makes both cases harder to change independently.

Occurrence count is a proxy for this question: at 3+ copies the change drivers are usually shared; at 2 copies they often diverge. Apply the count as a starting signal — if the change-driver answer is clear and unambiguous, it takes precedence over the count. A pure utility with no domain semantics shared across 2 call sites may justify extraction; domain logic appearing in 3 places but changing for different product reasons does not.

**Complexity placement**: If callers must know implementation details to use a function correctly, the abstraction boundary is in the wrong place — pull complexity downward into the module.

**Failure path ownership**: Name the recovery owner before adding error handling. The right owner is usually the outermost caller with full context — not the inner function that detected the problem. Scattered partial recovery across layers is worse than explicit propagation with a single owner.

**Contract change detection**: Callers depend on observed behavior, not documented behavior. Any change to default return values, side effects, or error types is a contract change — regardless of what the docs say.

**Dependency direction**: New imports crossing package or module boundaries are structural commitments. Before adding one, confirm the dependency flows from unstable toward stable code — not the reverse.

**State tracing**: Before any refactor that moves code across module boundaries, explicitly enumerate every piece of state the code reads or writes. Missing one creates a consistency window that did not exist before.

**Performance anti-patterns** — Check these regardless of whether the code is on a hot path; they are structural mistakes, not optimization choices:
- Query, HTTP call, or file read inside a loop → confirm a batch alternative exists or is not applicable before accepting [N+1 is the single most common performance regression and is always fixable without profiling]
- Collection loaded entirely into memory without a size bound → confirm it is bounded at production scale [unbounded loads cause OOM incidents, not slow responses]
- New external call (HTTP, database, subprocess) with no timeout → a timeout is required [no timeout converts one slow dependency into an indefinite caller freeze]
- Persistent mutable state in a long-running process that accumulates with each unit of work and has no eviction, rotation, or size bound → confirm the accumulated key space is bounded by design, or surface as unbounded memory growth [a continuously running process will exhaust memory in proportion to total work processed — this passes all tests and fails only under sustained production load]

If the code is on a hot path (request handlers, event processors, or any path that scales with data volume):
- Nested loops over a collection whose production size is unknown → flag as O(n²) risk and confirm n is bounded
- Sequential independent I/O calls → confirm they cannot be parallelized [sequential A→B→C when they are independent: latency = A+B+C; parallel: max(A,B,C)]

For detailed guidance per dimension, see `references/performance-dimensions.md`.

**Scope trigger**: If one judgment dimension raises a concern — scope a focused improvement. If multiple dimensions conflict — pause and make the conflict explicit. If none fire — narrow patch. For detailed scope decision support, see `references/risk-triage.md`.

## Completion Criteria

- [ ] The diff is scoped to what the task requires — no files, functions, or abstractions added beyond direct necessity.
- [ ] If the change touches shared state, reads and writes were traced across all affected boundaries.
- [ ] If the change requires migration (schema, API, config), deployment ordering, backward-compatible read/write behavior during rollout, and rollback or fallback impact were identified; any unknown was surfaced before implementation.
- [ ] If invoked after receiving code review, the implemented change addresses the review concern while preserving the existing responsibility owner, public contract, and dependency direction; any change to those was surfaced before implementation.
- [ ] If the change makes an existing capability path unreachable, that impact was surfaced explicitly before implementation.
- [ ] If implementing toward a goal identified by a reviewer, that goal was confirmed by the user rather than only inferred.
- [ ] Performance anti-patterns checked: no query/call-per-loop-item without confirming batching is not applicable, no collection loaded into memory without a confirmed production-scale bound, no new external call missing a timeout.
- [ ] If the code runs in a persistent process (server, event consumer, daemon), any persistent mutable state that grows with work volume has a confirmed bound or eviction mechanism, or the risk is surfaced explicitly.

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is artifact-type: completion is verified by self-checking the produced diff against the Completion Criteria checklist above. No separate user confirmation is required as a completion step — but the diff must be the evidence, not the agent's assessment of the diff.

## Anti-Rationalization Check

This section exists because a tidy-looking diff and a structurally sound implementation are not the same thing — and the agent cannot reliably distinguish them from the inside.

Pause before exiting.

Did I add anything justified only by "this supports testability," "this could be useful," or "this is clean design" — without a concrete caller or usage evidence in the existing code?

Before exiting, name the evidence used for soundness: affected callers checked, contracts preserved or surfaced, state touched traced, and verification run or intentionally skipped with reason. If no concrete evidence exists, do not treat the tidy diff as sufficient.

Completion-faking signals specific to implementation — stop if any apply:
- Tests were modified to make the change pass rather than because the behavioral contract changed — passing tests are not evidence of correctness if the tests were adjusted to fit the implementation
- An abstraction was introduced where no second concrete caller exists in the codebase at the time of writing
- Behavior changed under an existing function name without surfacing that the implicit contract changed
- A parameter or conditional was added to an existing abstraction instead of questioning whether the abstraction is still the right boundary
- The diff is scoped to what was asked without checking whether the ask itself points to the wrong layer

For detailed scope decision heuristics, see `references/risk-triage.md`.

## Self-Correction Signals

Stop and revise when:

- you are implementing directly from the request before reading the affected function, its callers, and the module boundary it belongs to
- you are treating the first editable location as the right implementation location
- you are adding parameters or conditionals to an existing abstraction before checking whether each branch maps to a distinct caller-specific behavior
- you are introducing an abstraction without a second concrete caller that demonstrates the need
- you are implementing accepted review feedback by copying the reviewer's suggested code or location without checking responsibility ownership, dependency direction, and public-contract impact
- you are treating unconfirmed scope or acceptance criteria as settled — if the user did not state it, it requires confirmation before implementing, not inference from the request wording
- you are skipping this skill because codebase investigation or root-cause analysis is complete — prior investigation confirms where to change, not whether the implementation is structurally sound
- you are making changes to adjacent code not required by the task rather than noting them for later
- you are using an external API, function signature, or library behavior from memory without verifying against actual source or documentation
- you have checked at least two applicable dimensions on a small change and found no observable contract, ownership, state, dependency, or performance blocker — stop analysis, pick the simpler local edit, and write the code
- you see a new query or external call inside a loop without confirming that batching is not possible
- you see a collection loaded entirely into memory without confirming its production-scale size bound
- you add a new external call without a timeout
- you identified a structural conflict (broken contract, state ownership ambiguity, wrong responsibility layer) and are producing code alongside the concern rather than stopping until the conflict is resolved
- the request uses simplicity framing ("one-liner," "quick fix," "straightforward") and you have not checked the affected call sites and contracts for evidence that contradicts that framing — the framing is the requester's prior, not your conclusion
- you are reviewing code in a long-running process and have not checked whether any persistent mutable state accumulates with work volume without a bound or eviction mechanism

---

For detailed reasoning heuristics on each structural concern, see `references/judgment-dimensions.md`.
For scope decision support, see `references/risk-triage.md`.
For code review response guidance, see `references/review-response-judgment.md`.
For AI-specific failure patterns and explanations, see `references/ai-agent-pitfalls.md`.
