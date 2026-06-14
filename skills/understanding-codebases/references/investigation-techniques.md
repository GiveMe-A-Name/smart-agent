# Investigation Techniques

Choose techniques by the missing working-model slice, not by habit. Each lookup must establish a relationship the implementation depends on: an entry path, behavior owner, data shape, consumer contract, local convention, verification gate, or unknown risk.

## Anchor on Entry Points

Use this when the working model does not yet explain how the relevant behavior is reached. Entry points vary by codebase type:
- **Web apps**: route handlers, API endpoints, middleware chains
- **CLI tools**: command definitions, argument parsers
- **Libraries**: exported public API, main module index
- **Event-driven**: event listeners, message handlers, webhook receivers

From the entry point, trace inward toward where decisions are made and state changes. The output is an entry path, not a change point.

## Trace the Behavior Owner

Use this when the current candidate file may only route, register, adapt, or export behavior. Trace through layers until you reach code that makes decisions or changes state — not just code that routes or configures.

Recognize and move past these delegation layers quickly:
- **Facades/wrappers**: re-export, rename, or add minimal logic
- **Configuration layers**: wire dependencies together but contain no business logic
- **Middleware/interceptors**: modify context but rarely own core behavior
- **Adapters**: translate between interfaces without adding decisions

Stop when you reach code that contains conditional logic, data transformations, state mutations, I/O calls, or contract enforcement relevant to the current task.

## Trace Data and State

Use this when the change depends on a value, object, record, state transition, or config-controlled branch. Trace:
- where the value comes from: API input, database row, file, environment, fixture, generated artifact, or caller
- its shape: type, schema, migration, validation rule, serializer, or fixture
- transformations: parsing, normalization, defaulting, filtering, mapping, persistence, or cache writes
- branch controls: flags, config values, permissions, enum states, and feature gates

Stop when the model can name the source, shape, relevant transformations, and the point where the value controls the behavior being changed.

## Analyze Consumers (Reverse Tracing)

Use this before changing shared code, public APIs, types, persisted data, config, or behavior that may have multiple callers. Trace who calls or consumes it:
- Use "Find References" / search for function/class name across the codebase
- Distinguish direct callers from transitive dependents
- Note whether the code is called from one place or many
- Identify assumptions consumers make about inputs, outputs, exceptions, ordering, side effects, or persistence

This is essential when evaluating whether a change point is safe — a function called from 15 places carries different risk than one called from 2.

## Discover Patterns and Conventions

Use this before adding a feature, new variant, handler, command, schema, event, integration, or test. Find how the codebase already does similar things:
- Search for analogous features (e.g., if adding a new API endpoint, find existing endpoints)
- Note naming conventions, file organization, registration paths, error handling, logging, and validation approaches
- Look at how tests and fixtures are structured for similar functionality
- Check for shared utilities, base classes, generators, or helper functions that analogous code already uses

When deviating from an existing pattern, name the observed pattern and the repository evidence that makes it unsuitable for the current change.

## Read Types, Interfaces, and Schemas

Use this when the data or contract model is unclear. Type definitions, interfaces, and schemas are the fastest way to understand a module's contract without reading every line of implementation:
- **Typed languages**: interface files, type definitions, abstract classes
- **API boundaries**: OpenAPI specs, GraphQL schemas, protobuf definitions
- **Database**: migration files, model definitions, schema files
- **Config**: config types, environment variable definitions

These tell you what data flows where and what each module promises to its consumers.

## Use Tests as Documentation

Use this when intended behavior, edge cases, fixtures, or verification path are unclear. Test files often reveal intended behavior more reliably than comments or docs:
- Test names describe expected behavior in concrete terms
- Test fixtures show realistic input/output examples
- Edge case tests reveal boundary conditions the original author considered
- Missing tests reveal areas of uncertainty or risk

When a module's purpose is unclear, its tests are often the best explanation. When tests are absent, record that absence as verification risk instead of assuming behavior from implementation alone.

## Consult History When "Why" Matters

Use this when code seems strange, over-engineered, contradictory, or when a behavior-preserving preparation may be needed before the intended change. The rationale is often in history:
- **git blame**: who wrote this line and when — follow to the commit message
- **Commit messages and PR descriptions**: explain the intent behind a change
- **Recent change frequency**: files changed often may be unstable or central
- **Large refactoring commits**: reveal architectural decisions and migration patterns

Use history to understand design decisions — not to audit code line by line.

## Read Directory Structure for Architecture

Use this when architecture, ownership, or entry-point discovery is blocking the working model. Spend a short pass understanding how the project is organized:
- Top-level directories often map to architectural layers or domains
- `package.json`, `Cargo.toml`, `go.mod`, or equivalent reveal dependencies and project shape
- Monorepo structures reveal module boundaries
- Shared/common directories reveal cross-cutting utilities

This pass must produce a concrete next lookup, such as an entry point, module boundary, or dependency to trace. Stop when the structure is no longer narrowing the current change or explanation task.
