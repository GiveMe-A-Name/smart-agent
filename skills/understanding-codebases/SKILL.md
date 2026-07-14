---
name: understanding-codebases
description: "Investigate a codebase's current design, runtime behavior, contracts, design intent, and reliable change points. Use when those facts have not yet been established from repository evidence for an implementation or explanation task."
---

# Codebase Understanding

Investigate the smallest relevant codebase slice needed to explain how the system is designed, how the target behavior occurs, and why the current structure may exist. Produce a current-system working model, not an implementation recommendation.

## Principles

- **Distinguish design, behavior, and intent.** Current design is the observed arrangement of modules, responsibilities, dependencies, data, and contracts. Runtime behavior is the execution or state path that produces the result. Design intent is the documented or historically supported reason for that arrangement. Do not collapse one into another.
- **Treat descriptions as hypotheses.** User explanations, issues, comments, filenames, and symbol names direct investigation but do not prove behavior or intent [because implementation and historical rationale can diverge from their labels].
- **Investigate from a decision-relevant uncertainty.** Each lookup should establish a relationship that could change the model of the relevant design, behavior, contract, or verification surface. Do not browse for general familiarity.
- **Trace to the behavior owner.** Follow entry points, wrappers, registries, adapters, and configuration until reaching the code that decides, transforms, mutates state, performs relevant I/O, or enforces the contract. An entry point proves reachability, not ownership.
- **Follow data and consumers when they constrain the model.** Trace the source, shape, transformations, persistence, overrides, and direct consumers only far enough to explain relevant branches and observed contracts.
- **Reconstruct intent from durable evidence.** Prefer ADRs, design docs, tests, schemas, commit or review history, and explanatory comments. Label intent as `documented`, `inferred`, `conflicting`, or `unknown`; current code shape alone does not prove why it was chosen.
- **Keep convention separate from recommendation.** Existing analogues reveal extension points, discovery gates, and local assumptions, but their existence does not prove that a future change should copy them.

## Working Model Constraints

Before concluding, establish the portions that matter to the task:

- the relevant module or responsibility boundary in the current design;
- at least one concrete execution, data, or state path to the current behavior owner;
- caller-observable contracts and affected consumers when shared behavior is involved;
- the available verification surface, or the absence of one;
- any design-intent evidence that could explain a non-obvious boundary or constrain a future change;
- unknowns that could materially change one of the above.

Do not force every slice into every task. Deepen a slice only when a different finding could change the identified owner, architecture relationship, contract, design intent, or verification surface. Stop when remaining unknowns cannot change that model; name material unknowns rather than filling them with inference.

The output should separate `current design`, `runtime cause`, `design intent`, `contracts`, `verification`, and `unknowns` when those distinctions matter. It may identify the current behavior owner and evidence-backed candidate change points. Do not choose the final placement, local patch, abstraction, refactor, or migration; those are future-design decisions rather than facts about the current system.

Read `references/working-model-slices.md` when deciding which evidence categories matter to the current task.
Read `references/investigation-techniques.md` when a specific design, behavior, consumer, data, or intent relationship is still untraced.
Read `references/task-driven-investigation.md` when the task type is known but the relevant working-model slice is unclear.
Read `references/challenging-situations.md` when dynamic wiring, missing tests, poor naming, missing intent records, or codebase size blocks a reliable model.

Example index:

| Example | Read when |
|---|---|
| `examples/tracing-behavior.md` | Evidence stops at an entry point or wrapper. |
| `examples/pattern-discovery.md` | A feature depends on registration, discovery, or local conventions. |
| `examples/bug-fix-working-model.md` | A bug depends on data shape, consumers, configuration, or tests. |
