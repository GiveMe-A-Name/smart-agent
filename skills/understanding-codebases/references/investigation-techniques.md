# Investigation Techniques

Choose a technique for a missing relationship in the working model. Each lookup should confirm, revise, or bound a claim.

## Map the current design

Use package manifests, directory structure, public exports, interfaces, schemas, and dependency wiring to identify module boundaries and dependency direction. Treat names and folder layout as hypotheses until imports, construction, or runtime wiring confirms them.

## Trace one runtime path

Start from a concrete route, command, public API, event, job, or test. Follow delegation until the relevant decision, mutation, or side effect occurs. Move quickly past wrappers, registries, adapters, and configuration unless they enforce a contract relevant to the behavior.

## Trace data and state

Follow behavior-controlling values from source through validation, defaults, transformation, persistence, caching, and branch conditions. Read types, schemas, migrations, serializers, configuration definitions, and fixtures when they define the real contract.

## Reverse-trace consumers

Search references and implementations before concluding that a boundary is local. Record assumptions about inputs, outputs, errors, timing, ordering, side effects, persistence, and identity. For indirect wiring, inspect registries, annotations, generated manifests, dependency injection, and feature flags.

## Discover conventions

Trace an analogous feature end to end, including registration and tests. Distinguish a required discovery mechanism from a common style. Existing patterns describe the repository; they do not by themselves prove the right future design.

## Reconstruct design intent

Use evidence in this order when available:

1. ADRs, design documents, and explicit architectural comments.
2. Tests, schemas, and compatibility constraints that encode deliberate behavior.
3. PR descriptions, commit messages, and blame-linked history.
4. Repeated structure or naming, which supports only an inference.

Record the claim and its strength: `documented`, `inferred`, `conflicting`, or `unknown`. Current code proves what it does, not why its authors chose it.

## Use tests and history precisely

Tests are evidence of exercised contracts, not proof that every behavior is intentional. History is useful when a line or structure appears constrained or surprising; targeted blame and commits are preferable to reading chronology broadly. Missing tests or rationale should remain explicit evidence gaps.
