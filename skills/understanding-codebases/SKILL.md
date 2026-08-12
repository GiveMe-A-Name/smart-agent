---
name: understanding-codebases
description: "Investigate a codebase's current design, runtime behavior, contracts, design intent, and reliable change points. Use when those facts have not yet been established from repository evidence for an implementation or explanation task."
---

# Codebase Understanding

Investigate the smallest relevant codebase slice needed to explain how the system is designed, how the target behavior occurs, and why the current structure may exist. Produce a current-system working model, not an implementation recommendation.

## Keep Different Claims Separate

- **Current design** is the observed arrangement of responsibilities, boundaries, dependencies, data, and contracts.
- **Runtime behavior** is the execution, data, or state path that produces an observed result.
- **Design intent** is the documented or historically supported reason for that arrangement.

Do not infer one from another. User descriptions, issues, comments, filenames, and symbol names are leads, not proof. Current code can establish what the system does, but code shape alone cannot establish why it was chosen.

## Build the Working Model

Start from the decision-relevant uncertainty in the task. Investigate only relationships whose outcome could change the behavior owner, architecture relationship, contract, impact boundary, design intent, or verification surface.

1. **Bound the task.** Identify the behavior being investigated and separate it from adjacent behavior that does not need to be explained.
2. **Trace one concrete path.** Begin at a relevant route, command, public API, event, job, test, or data source. Follow delegation through wrappers, adapters, registries, and configuration until reaching the code or boundary that decides, transforms, mutates state, performs the relevant I/O, or enforces the contract.
3. **Locate the behavior owner.** An entry point proves reachability, not ownership. For example, in `route -> service -> policy`, the route may expose the behavior while the policy owns the decision.
4. **Trace constraints around the owner.** Follow behavior-controlling values through their sources, defaults, validation, transformations, persistence, and branch conditions when they affect the result. Reverse-trace direct consumers when shared behavior is involved, recording observable assumptions about inputs, outputs, errors, timing, ordering, identity, persistence, or side effects.
5. **Recover intent only when it matters.** Consult intent evidence when a boundary is surprising, historically constrained, or relevant to a later responsibility decision. Prefer ADRs and design documents, then tests and schemas, then targeted commit or review history and explanatory comments. Label intent as `documented`, `inferred`, `conflicting`, or `unknown`.
6. **Identify verification and unknowns.** Find the tests, checks, logs, or reproduction paths that expose the current behavior. If none exists, state the verification gap. Name missing evidence that could materially change the working model instead of filling it with speculation.

## Adjust the Investigation to the Risk

- For a bug, find the first divergence between the observed path and the supported contract; account for data, configuration, state, and shared consumers that could produce the same symptom.
- For an extension, trace one analogue end to end and distinguish required discovery mechanisms from repeated style. A new event file is not reachable if a registry, generated manifest, annotation scan, dependency-injection binding, package entry, or feature flag must expose it.
- For refactoring context, establish preserved contracts, callers, side effects, ordering, state ownership, and the verification surface without deciding whether or how to refactor.
- For an architecture explanation, trace a representative path across boundaries and explicitly bound important variants that were not traced.

Ordinary reference search is incomplete in dynamically wired or generated systems. Missing tests limit verification; they do not make unobserved behavior verified. When documentation, tests, code, and history disagree, report the conflict and its scope rather than choosing the most convenient story. Existing analogues reveal conventions and extension gates, but do not prove that a future change should copy them.

## Completion Boundary

Before concluding, establish the relevant portions of:

- current responsibility boundaries and dependency direction;
- a concrete path to the behavior owner;
- behavior-controlling data or state;
- caller-observable contracts and affected consumers;
- supported design intent;
- verification surfaces and material unknowns.

Do not force every category into every task. Stop when remaining unknowns cannot change the owner, architecture relationship, contract, intent, impact boundary, or verification surface. Present the categories distinctly when combining them would blur the evidence, but do not force a fixed output template.

The working model may identify an evidence-backed reliable change point. Do not choose the final patch location, abstraction, refactor, migration, or dependency redesign; those belong to subsequent implementation judgment.
