---
name: implementation-judgment
description: "Choose a structurally sound change shape from an evidence-backed working model. Use before a bug fix, feature, or refactor that can affect runtime behavior, contracts, responsibilities, abstractions, or dependencies; skip confirmed non-behavioral edits."
---

# Implementation Judgment

Use an evidence-backed working model of the current system to decide how a behavior change should land. The target is not the smallest diff or the cleanest imaginable architecture. Choose a design that fully carries the current requirement without degrading the relevant structure, while keeping unrelated redesign out of scope.

## Principles

- **Separate current-system evidence from future design.** Treat the working model as evidence of how the system is designed, how it behaves, and why it may have evolved that way; use it as input rather than assuming the current behavior owner must remain the future owner.
- **Keep quality above the floor and scope below the ceiling.** The change must not buy a small diff with misplaced responsibility, duplicated policy, caller-specific branches, hidden contract changes, or wrong-way dependencies. Stop once the requirement has a coherent owner and the structural pressure introduced or exposed by this change is resolved [because adjacent redesign adds risk without helping the current outcome].
- **Put responsibility where the complete decision can be made.** Parsing belongs at an input boundary, domain policy with domain context, persistence invariants with the state owner, and recovery with the caller that can choose retry, rollback, user response, or alerting [because splitting one concern across partial owners produces inconsistent behavior].
- **Preserve observed contracts deliberately.** Return values, errors, side effects, persistence, events, ordering, timing, caching, and identity are contracts when callers can observe them. Preserve them, change them explicitly, or provide a staged compatibility path.
- **Abstract shared knowledge, not surface similarity.** Create a shared abstraction when current consumers must obey the same rule and change for the same reason. Keep similar-looking code separate when its policy, lifecycle, security boundary, or owner differs; a single-use helper may name or decompose behavior without pretending to be a reusable extension point.
- **Use preparatory refactoring when the current structure obstructs a clean change.** Make a bounded behavior-preserving structural move that creates the needed seam or owner, verify it, then add the behavior. Use a focused structural change when no such preparation can prevent the feature from deepening a known boundary problem.
- **Treat verification as design evidence.** Tests support the chosen shape only when they still assert caller-observable behavior; do not weaken tests, snapshots, types, lint rules, or verification scope to make an unsuitable design pass.

## Decision Constraints

Before choosing an intervention, establish the requested behavior, caller-visible behavior that must be preserved, and available acceptance evidence. If materially different outcomes would all satisfy the wording, or no observable result distinguishes success, clarify the requirement before designing the change. Treat user-stated non-goals as scope constraints.

Before writing behavior-changing code, choose and be able to justify one intervention:

- **Local change** only when the behavior fits the existing owner and does not add split responsibility, duplicated policy, a caller-specific switch, a wrong-way dependency, or an unacknowledged contract change.
- **Preparatory refactoring** when a specific existing structure blocks the clean behavior change and a bounded, behavior-preserving move can remove that obstruction.
- **Focused structural change** when every local option would violate an ownership, abstraction, contract, dependency, or state boundary. Limit it to the boundary the current requirement actually exercises.
- **Staged change** when compatibility, persisted data, deployment order, or rollback requires old and new shapes to coexist.
- **Pause** when the working model cannot establish the relevant owner, caller contract, state source, or migration order. Name the missing evidence that could change the design decision; do not restart a broad repository investigation.

When a change adds or alters an external entry point, identity or permission flow, sensitive-data path, secret handling, or dependency across a trust boundary, identify who authenticates the actor, where authorization is enforced, what data crosses the boundary, and which side validates it. Do not choose a design that relies on client-side checks, caller-supplied identity, implicit trust, or broader data or dependency access than the requirement needs.

Do not generalize from a hypothetical future consumer, extract an abstraction only to remove visual duplication, or expand the change to repair unrelated debt. When the chosen design materially affects the implementation, state the owner, contract treatment, structural intervention, and excluded adjacent work.

Read `references/judgment-dimensions.md` when the change presents a concrete ownership conflict, strained abstraction, shared contract change, trust-boundary or sensitive-data change, cross-boundary dependency, persistent state transition, or migration.
Read `references/performance-dimensions.md` when the proposed shape adds I/O, work inside a growing loop, persistent resources, database access, caching, or concurrency.

Example index:

| Example | Read when |
|---|---|
| `examples/wrong-abstraction.md` | Similar code may represent shared knowledge or independent policies. |
| `examples/complexity-misplacement.md` | One validation, transformation, or recovery concern is split across layers. |
| `examples/implicit-contract-break.md` | A signature-safe refactor may change caller-observable behavior. |
