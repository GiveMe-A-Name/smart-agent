# Challenging Codebase Situations

These situations make it easier for an agent to over-read, under-read, or invent a model. Use the observable signals below to keep the working model useful for implementation.

## Large Codebases — Knowing When "Enough" Is Enough

In a large codebase (hundreds of thousands or millions of lines), you cannot understand everything before acting. The skill is progressive comprehension — building enough of the current task's working model to edit and verify safely, then deepening as needed.

**Key signals**:
- **Start narrow, widen on evidence.** Begin with the smallest slice that could support the current change or explanation. Only expand scope when the current slice reveals dependencies you cannot ignore. "Let me understand the whole system first" is not a strategy — it's procrastination.
- **Use module boundaries as natural stopping points.** If your task is within one module, you need to understand that module's contract (interfaces, types, public API) and its immediate dependencies — not the entire dependency tree. Trust the abstraction boundary until evidence suggests it's leaky.
- **Stop on named sufficiency, not feeling.** Stop expanding only when you can name the change point, entry/data path, convention or analogue, consumer risk, verification path, and non-critical unknowns. "This feels enough" is not a checkable stopping condition.
- **Track what you've explored and what you haven't.** In a large codebase, maintain a map of: explored and understood, explored but uncertain, known to exist but not explored, unknown.

## Poorly Documented or Poorly Named Codebases

When variable names are cryptic, comments are absent or misleading, and no documentation exists, standard investigation techniques still work — but require more inferential reasoning.

**Key signals**:
- **Tests are the most reliable documentation.** When code is poorly documented, test files reveal intended behavior through concrete examples. Test names (even bad ones) hint at what the author thought was important. Test fixtures show realistic data shapes.
- **Git history compensates for missing comments.** When a function is named `processData` and does something non-obvious, `git blame` and the corresponding commit message or PR description often explain what the original author intended. Large refactoring commits reveal architectural intent.
- **Trace data flow when names are unhelpful.** When `x` is passed to `handleThing` which calls `doStuff`, trace the actual data: where does `x` come from (API input? database record? config?), what type is it, what transformations does it undergo, where does the result go? Data flow reveals purpose even when names don't.
- **Look for the domain model.** Even in poorly named code, there is usually a core set of entities and their relationships. Find the database schema, the protobuf definitions, or the type declarations — these reveal the domain model that the application code operates on. Understanding the domain model makes the code comprehensible even when individual names are opaque.
- **Do not rename or refactor to understand.** It's tempting to rename variables while exploring to make code clearer. Resist this — renaming before understanding risks encoding wrong assumptions. Understand first, rename later (if the task calls for it).

## No Clear Analogue

When search finds no similar feature, do not treat the absence as permission to invent any structure.

**Key signals**:
- **Check adjacent analogues.** Look for similar registration, validation, persistence, error handling, or test patterns even when no feature has the same domain shape.
- **Use contracts as the fallback.** If there is no analogue, use public interfaces, schemas, package boundaries, and tests to choose a shape that fits existing contracts.
- **Record the missing precedent.** Note that no direct analogue was found and name the closest patterns used instead, so the deviation is explicit rather than accidental.

## Dynamic or Indirect Wiring

Reflection, dependency injection, generated code, plugin registries, feature flags, and config-driven wiring can hide the real execution path.

**Key signals**:
- **Find the registration mechanism.** Search for registry entries, annotations, decorators, config keys, generated manifests, or provider bindings before assuming a direct caller search is complete.
- **Check generated and package configuration.** If files are discovered by naming convention, codegen, package metadata, or build config, read that discovery mechanism before adding files.
- **Trace one concrete instance.** Pick an existing registered item and follow how it is loaded, instantiated, tested, and invoked; use that instance as the map for the current change.

## Missing or Weak Tests

When tests are absent, stale, or too broad, the verification model is incomplete.

**Key signals**:
- **Use tests to locate intended behavior, not to excuse missing coverage.** Existing tests can still reveal fixtures and edge cases even when they do not cover the current path.
- **Name the verification gap.** If no local test can fail before the change, record the missing test or manual check instead of reporting the code as verified.
- **Prefer a focused characterization test before risky edits.** When behavior must be preserved and no test captures it, add or identify a small check around the current contract before refactoring.
