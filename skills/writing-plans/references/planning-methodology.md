# Planning Methodology

Reference material for decomposition, contingency, revision triggers, stop signals, and scope control. `SKILL.md` owns the assessment gate and hard constraints; this file handles cases where task boundaries or execution controls are not obvious.

If the plan cites a file or pattern not yet read, mark it as an assumption and verify it before dependent tasks. Do not present inferred repository structure as current-state evidence.

---

## How to Think About Decomposition

There is no single decomposition recipe. Use the failure or learning boundary that matters for the specific change.

**Create an end-to-end behavior before expanding layers.** The first or second task should connect the smallest usable path through the system, even with hardcoded data, when the feature spans multiple components. If all early tasks create models, infrastructure, or schemas without a runnable behavior, the plan delays discovery until too late.

**Move critical uncertainty early.** Put a task near the front when it depends on an unverified integration, an API capability not yet observed, a performance/security constraint, or a user decision. If that task fails, downstream tasks built on the assumption would be wasted.

**Expose dependency edges.** Mark tasks as parallel only when they do not modify the same files, do not consume each other's output, and share an already defined interface.

**Define contracts before parallel cross-boundary work.** If the plan crosses module, service, or team boundaries, add an interface/contract task before work that depends on both sides. The contract must name inputs, outputs, errors, and ownership/handoff point.

**Split or merge by verification size.** Split a task if it has multiple independent verification commands, modifies unrelated behavior paths, or cannot be summarized by one purpose sentence. Merge tasks when each one has no independent behavior, uncertainty reduction, or verification signal.

---

## Contingency Planning and Plan Revision

Use the approval-surface test in `SKILL.md` before choosing a response. The cases below help when it is not immediately clear whether execution may adapt or must stop.

**Plan-internal response.** A contingency or revision trigger may change task detail, file ownership inside an approved boundary, or the choice between already approved implementation paths. For example:
- a spike rules out one approved sandbox mechanism, so execution uses the other approved mechanism
- an observed data shape changes a private adapter but preserves the approved contract
- the first tasks show that two later tasks should be merged without changing outcome, scope, or design semantics

**Approval-changing response.** Use a stop signal instead when the only viable response changes the Decision Brief or Design Review. A new requirement, incompatible public API, required migration, or invalidated ownership boundary is not a plan revision the executor may approve autonomously.

**Cross-boundary coordination.** When the plan touches code owned by others (different modules, different teams, shared interfaces):
- **Define the contract first.** If building two sides of an interface, agree on it before building either side.
- **Explicit handoff points.** Where does one piece of work end and another begin? What does "done" look like for the piece you own? Ambiguous boundaries cause duplicate work or gaps.

---

## Stop Signals

A stop signal is a pre-declared condition under which the agent must pause and ask before continuing because the response would change the approved surface. Stop signals are not failure conditions; they are expected forks where the right path requires human judgment.

Two categories cover most approval-sensitive cases:

**Resource change** — the plan is about to introduce something that wasn't agreed to:
- "If resolving this requires a new external dependency, pause and confirm before introducing it."
- "If the fix requires changing the public API contract, pause before proceeding."

**Approved premise invalidated** — an assumption behind the approved outcome or design turns out to be false:
- "If imported records lack the stable identifiers required for idempotency, pause before designing a workaround."
- "If the existing schema doesn't support this, pause — the plan assumed it did."

Write stop signals as concrete conditions, not vague thresholds. "If something seems complex" is not a stop signal. "If adding retry logic requires a new external dependency" is. If disproving an assumption only changes unapproved task detail, keep it as a revision trigger instead.

---

## Explicit Exclusions and Conflict Handling

**Explicit exclusions.** State what the plan deliberately does not include. This is most valuable when the scope boundary is non-obvious — when a reasonable agent could plausibly infer that X is in scope, even though it isn't.

- "Not building an administration UI — configuration remains CLI-only."
- "Not handling partial failures — assumes all-or-nothing semantics."
- "Not adding authentication to the new endpoint in this iteration."

One or two lines is enough. The goal is to prevent execution-time scope expansion, not to enumerate everything that won't be built.

**Conflict handling.** When the plan has multiple goals that can pull in opposite directions, name the tiebreaker explicitly. An agent that hits a real conflict mid-execution will make the call itself if no priority is declared — and it will often get it wrong.

Format: `When [X] conflicts with [Y], prefer [X]. [One sentence of reasoning.]`

Examples:
- "When correctness conflicts with performance, prefer correctness. Performance can be improved post-launch."
- "When security conflicts with schedule, prefer security. Flag the blocker rather than working around it."
- "When completeness conflicts with code cleanliness, prefer completeness in this iteration — refactoring is a separate task."

If the tiebreaker changes outcome, compatibility, risk, or scope, put the proposed choice in **Approval Needed**. If it is an execution constraint within an already approved outcome, put it before the task list. Do not create a standalone approval field merely because two implementation preferences compete.
