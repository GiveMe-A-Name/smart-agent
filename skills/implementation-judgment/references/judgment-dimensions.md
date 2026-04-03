# Judgment Dimensions

When reasoning about how a change should land, work through these dimensions. Not every dimension applies to every change — but skipping a relevant one is how subtle damage happens.

## 1. Abstraction Correctness

**Question**: Are the existing abstractions helping or hurting? Should this change create, extend, preserve, or undo an abstraction?

**Key signals**:
- If an abstraction requires parameters and conditional branches to handle different cases, it may be the wrong abstraction. The fastest way forward is often back — inline the abstraction, let the callers diverge, then see what new pattern emerges.
- If code is duplicated in 2 places and the copies are diverging, keep them separate. Premature unification creates a coupling point that makes both cases harder to change.
- If code is duplicated in 3+ places with identical semantics and the same rate of change, extraction is likely warranted.
- If you're about to "clean up" repetition, ask: are these cases truly the same, or do they just look the same right now? Code that looks similar but changes for different reasons should not share an abstraction.
- If an existing shared abstraction has accumulated conditionals to handle special cases, consider whether inlining it back would make each caller simpler and more honest about what it actually does.

See `examples/wrong-abstraction.md` for a case where eagerly extracting surface-similar code creates a worse outcome than tolerating duplication. See `examples/security-review-duplicate-helper.md` for the opposite case — where 3+ identical copies of security logic should be consolidated into a shared helper.

## 2. Complexity Placement

**Question**: Which layer should own this complexity? Is the change pushing complexity toward the right boundary or scattering it?

**Key signals**:
- Deep modules are better than shallow modules. A module with a simple interface that hides complex implementation is more valuable than one with a complex interface and trivial implementation.
- If callers need to know implementation details to use a function correctly, the abstraction boundary is in the wrong place.
- Pull complexity downward: if you can make 5 callers simpler by making 1 implementation more complex, do it. The net cognitive load decreases.
- When adding a configuration parameter or flag, ask: could this decision be made automatically inside the module instead of pushed to every caller? Each configuration parameter is complexity exported to every user.
- Validation, transformation, and error handling for a particular concern should live together in one layer, not be scattered across multiple layers with each layer doing a partial job.

See `examples/complexity-misplacement.md` for a case where partial validation spread across layers creates a false sense of safety.

## 3. Failure Mode Awareness

**Question**: How will this code fail? Are the failure paths explicit and handled at the right level?

**Key signals**:
- Error handling is most effective at the ends (end-to-end principle). If every intermediate layer handles errors partially, you end up with duplicated, inconsistent recovery logic and errors that get swallowed or transformed beyond recognition.
- When adding error handling, ask: who is the right party to make the recovery decision? Often it is the outermost caller who has the full context, not the inner function that detected the problem.
- Distinguish between errors that should be retried, errors that should be surfaced to the user, and errors that indicate programmer mistakes. These require different handling strategies — conflating them leads to both swallowed bugs and noisy false alarms.
- When a function can fail in ways the caller doesn't check, the code has an implicit failure path. Make failure paths explicit, even if the handling is "propagate to the caller."
- Consider partial failure: if this operation does steps A, B, C and fails at B, what state is the system in? Is that state recoverable?

## 4. Deletability and Replaceability

**Question**: If requirements change, how easy is it to delete or replace this code? Is the change making the codebase more or less entangled?

**Key signals**:
- Code that is easy to delete is better than code that is easy to extend. Extension assumes you know the future; deletion works regardless.
- Isolate the parts that are likely to change from the parts that are likely to stay stable. Business rules change fast; data structures change slowly; protocols change slowest.
- If deleting this code would require changes in many unrelated modules, the coupling is too tight. Loose coupling means you can change your mind without rewriting the world.
- Feature flags, runtime configuration, and clear module boundaries all increase deletability. Deep inheritance hierarchies, global state, and implicit dependencies decrease it.
- When building something new and uncertain, favor a "big lump" that can be deleted as a whole over many small interlocking pieces that can only be deleted piecewise.

## 5. Contract and Compatibility Awareness

**Question**: What does this change mean for callers? Are implicit contracts being preserved or silently broken?

**Key signals**:
- Callers depend on observed behavior, not documented behavior. If a function has always returned an empty array on error (even by accident), callers may rely on that. Changing it to throw is a breaking change regardless of what the docs say.
- Adding a parameter with a default value looks safe but can still change behavior for callers who relied on the old default path through the code.
- Side effects are implicit contracts. If a function has always logged, written to a file, or emitted an event, removing that side effect can break callers who depend on it even though it was never part of the explicit API.
- When refactoring, hold the behavioral boundary constant even if the implementation changes completely. Tests that verify behavior (not implementation) are the safety net here.
- For public APIs and shared libraries, backwards compatibility is almost always more important than internal cleanliness. Ugly-but-compatible beats clean-but-breaking.

See `examples/implicit-contract-break.md` for a case where a "safe" refactoring breaks callers through implicit behavioral changes.

## 6. Dependency Direction

**Question**: Does this change make the dependency graph simpler or more complex? Are dependencies flowing in the right direction?

**Key signals**:
- Dependencies should flow from unstable (frequently changing) code toward stable (rarely changing) code. If stable infrastructure code depends on volatile business logic, the direction is wrong.
- A new import or dependency crossing a package/module boundary is a stronger signal than one within the same module. Cross-boundary dependencies are harder to undo.
- Circular dependencies are a structural problem, not just a lint warning. They mean two modules cannot be understood, tested, or deployed independently.
- When tempted to add a dependency "just for one function," consider the transitive cost: you're also depending on everything that function depends on.
- If two modules keep needing to change together, they are coupled regardless of whether there is an explicit dependency between them. Consider whether they should be merged or whether a missing abstraction needs to be introduced between them.

## 7. Performance Awareness

**Question**: What is the performance impact of this change? Is the change respecting the performance boundaries of the system?

**Key signals**:
- Hot path vs cold path: Is this code on a hot path? Changes on hot paths deserve more scrutiny for performance.
- Algorithmic complexity: Does this change introduce O(n²) or worse complexity where O(n) or O(n log n) was expected? Does it add work inside a loop that could be done outside?
- I/O on the critical path: Does the change add database queries, network calls, or disk I/O to a path that was previously CPU-only? Every I/O call is a potential latency cliff.
- N+1 patterns: Does the change query a resource per item in a collection when a batch query would serve? This is the single most common performance regression in database-backed systems.
- Memory allocation patterns: Does the change create objects in a tight loop, build unbounded collections, or hold references that prevent garbage collection?
- Caching and memoization: If the same computation is performed repeatedly with the same inputs, should it be cached? Conversely, does a cache introduced here create a stale-data risk that outweighs the performance gain?
- The premature optimization trap: Not every path needs optimization. If performance impact is speculative, measure before acting. But if the change is on a known hot path or introduces a known anti-pattern (N+1, unbounded collection, blocking I/O in async context), flag it — that is not premature, it is informed.

## 8. Observability and Operability

**Question**: When this code fails in production, will the person debugging it at 3am have the information they need?

**Key signals**:
- Structured logging: Does the change add, preserve, or degrade logging at key decision points? Log entries should carry enough context (request ID, user ID, relevant identifiers) to reconstruct what happened without reproducing the issue.
- Error context propagation: When errors cross boundaries (function → caller → API → user), is enough context preserved? An error message that says "operation failed" helps nobody. An error that says "failed to update user 12345: database connection timeout after 30s" enables diagnosis.
- Metrics and alerting surface: If this change introduces a new failure mode, is there a way to detect it through metrics? A silent failure that accumulates over days is worse than a loud crash.
- Health checks and readiness signals: If the change affects system readiness (new dependency, new resource requirement), does the health check reflect this?
- Debuggability under pressure: Could someone unfamiliar with this code, reading only logs and metrics, understand what happened? The audience is a tired on-call engineer, not the original author.

## 9. Security Posture

**Question**: Does this change maintain or improve the system's security posture? What attack surface does it introduce?

**Key signals**:
- Input validation: Does the change handle untrusted input? All data from external sources (user input, API responses, file contents, environment variables) must be validated before use. Validation should happen at the boundary, not scattered across layers.
- Authentication and authorization: If the change exposes a new endpoint, route, or data path, are auth checks in place? Check that authorization is enforced at the right layer — not just at the UI, but at the API/service layer.
- Data exposure: Does the change risk exposing sensitive data in logs, error messages, API responses, or URLs? PII, credentials, tokens, and internal system details should never appear where they shouldn't.
- Dependency trust: If the change adds a new dependency, what is the trust level? Evaluate the dependency's maintenance status, security history, and the blast radius of a supply-chain compromise.
- Principle of least privilege: Does the change request only the permissions it needs? Over-privileged code is a larger blast radius when compromised.
- Cryptographic hygiene: If the change involves encryption, hashing, or token generation, are current algorithms and practices used? Hardcoded secrets, weak hashing (MD5/SHA1 for security), and custom crypto are red flags.

## 10. State Management and Data Integrity

**Question**: Does this change correctly manage shared state? Will state transitions remain consistent under all conditions — including concurrent access, partial failure, and restart?

**Key signals**:
- **Shared mutable state is the most common source of subtle bugs in refactoring.** When moving code between components, trace every piece of state the code reads or writes. If a refactor splits a function that previously managed state atomically into two functions that now manage it across a boundary, you have created a consistency window that didn't exist before.
- **Identify the source of truth.** For every piece of state this code touches, there should be exactly one authoritative source. If data is denormalized or cached, the code must handle staleness explicitly — not assume freshness. When the same data exists in two places, ask which one wins on conflict and what happens when they disagree.
- **State transitions should be explicit and enforceable.** If an entity can be in states A, B, C, the valid transitions (A→B, B→C, but not A→C) should be enforced in code, not implied by call ordering. The question is not "does the current code path follow the right order?" but "what prevents a future caller from violating the order?"
- **Crash recovery determines real data integrity.** If the system fails mid-operation, what state is persisted? Any operation that modifies persistent state in multiple steps either needs atomicity (transaction) or idempotent recovery (can be safely re-run).
- **Local state and remote state will diverge.** When integrating external services, a successful API call followed by a local crash means the remote side has progressed but your system doesn't know. Design for this — idempotency keys, reconciliation jobs, or explicit "pending" states.
- **Schema migrations create a dual-state window.** Any migration that changes schema while the application is running creates a window where old code reads new schema (or vice versa). Ensure migrations are backward-compatible with the currently deployed code, or coordinate deployment order explicitly.

## 11. Migration and Backward Compatibility

**Question**: Does this change require a migration — of data, configuration, API consumers, or deployment sequence? Is the migration safe to run in production?

**Key signals**:
- Schema migrations that rename or remove columns while the application is running will break queries in the old code. Safe migrations add first, backfill, deploy new code, then remove old columns in a subsequent migration. A single migration that both adds and removes is a production risk.
- API changes that modify response shapes, remove fields, or change error formats break existing consumers. Even "internal" APIs may have consumers you don't know about — scripts, cron jobs, partner integrations. Treat all API changes as potentially breaking unless proven otherwise.
- Configuration changes that rename environment variables, change default values, or alter config file structure can break deployments silently. The application starts, but behaves differently in ways that won't surface until a specific code path is hit.
- Feature flags and gradual rollout reduce blast radius. When the change is risky or affects a large user base, consider a flag that allows rolling back without a deploy.
- Deployment ordering matters. If this change requires "deploy service A before service B," document that explicitly. Implicit deployment ordering is a deployment outage waiting to happen.
