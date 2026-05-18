# Investigation Techniques

These are techniques to select from based on the task — not a fixed sequence. Choose the ones that answer the current question most efficiently.

## Anchor on Entry Points

Start from where behavior begins. Entry points vary by codebase type:
- **Web apps**: route handlers, API endpoints, middleware chains
- **CLI tools**: command definitions, argument parsers
- **Libraries**: exported public API, main module index
- **Event-driven**: event listeners, message handlers, webhook receivers

From the entry point, trace inward toward where decisions are made and state changes.

## Trace the Delegation Chain

Most entry points delegate to deeper layers. Trace through the layers until you reach code that makes decisions or changes state — not just code that routes or configures.

Recognize and move past these delegation layers quickly:
- **Facades/wrappers**: re-export, rename, or add minimal logic
- **Configuration layers**: wire dependencies together but contain no business logic
- **Middleware/interceptors**: modify context but rarely own core behavior
- **Adapters**: translate between interfaces without adding decisions

Stop when you reach code that contains conditional logic, data transformations, or state mutations relevant to your question.

## Analyze Consumers (Reverse Tracing)

When you need to understand a piece of code's impact or role, trace who calls it:
- Use "Find References" / search for function/class name across the codebase
- Distinguish direct callers from transitive dependents
- Note whether the code is called from one place (likely safe to change) or many (change requires broader understanding)

This is essential when evaluating whether a candidate change point is the right one — a function called from 15 places carries different risk than one called from 2.

## Discover Patterns and Conventions

Before proposing how to implement something, find how the codebase already does similar things:
- Search for analogous features (e.g., if adding a new API endpoint, find existing endpoints)
- Note naming conventions, file organization patterns, error handling approaches
- Look at how tests are structured for similar functionality
- Check for shared utilities, base classes, or helper functions that your code should use

Following existing patterns is usually the right default. Deviating should be a conscious, justified decision — not ignorance.

## Read Types, Interfaces, and Schemas

Type definitions, interfaces, and schemas are the fastest way to understand a module's contract without reading every line of implementation:
- **Typed languages**: interface files, type definitions, abstract classes
- **API boundaries**: OpenAPI specs, GraphQL schemas, protobuf definitions
- **Database**: migration files, model definitions, schema files
- **Config**: config types, environment variable definitions

These tell you what data flows where and what each module promises to its consumers.

## Use Tests as Documentation

Test files often reveal intended behavior more reliably than comments or docs:
- Test names describe expected behavior in concrete terms
- Test fixtures show realistic input/output examples
- Edge case tests reveal boundary conditions the original author considered
- Missing tests reveal areas of uncertainty or risk

When a module's purpose is unclear, its tests are often the best explanation.

## Consult History When "Why" Matters

When code seems strange, over-engineered, or contradictory, the answer is often in its history:
- **git blame**: who wrote this line and when — follow to the commit message
- **Commit messages and PR descriptions**: explain the intent behind a change
- **Recent change frequency**: files changed often may be unstable or central
- **Large refactoring commits**: reveal architectural decisions and migration patterns

Use history to understand design decisions — not to audit code line by line.

## Read Directory Structure for Architecture

When architecture, ownership, or entry-point discovery is part of the current question, spend a short pass understanding how the project is organized:
- Top-level directories often map to architectural layers or domains
- `package.json`, `Cargo.toml`, `go.mod`, or equivalent reveal dependencies and project shape
- Monorepo structures reveal module boundaries
- Shared/common directories reveal cross-cutting utilities

This pass should produce a concrete next lookup, such as an entry point, module boundary, or dependency to trace. Stop when the structure is no longer narrowing the current question.
