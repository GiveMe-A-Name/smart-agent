# Judgment Dimensions

When reasoning about how a change should land, work through these dimensions. Not every dimension applies to every change — but skipping a relevant one is how subtle damage happens.

## 1. Abstraction Correctness

**Question**: Are the existing abstractions helping or hurting? Should this change create, extend, preserve, or undo an abstraction?

**Key signals**:
- If an abstraction requires parameters and conditional branches to handle different cases, it may be the wrong abstraction. The fastest way forward is often back — inline the abstraction, let the callers diverge, then see what new pattern emerges.
- Before extracting any duplication, ask: "What event would require me to modify each copy?" If every copy has the same answer, extraction reduces drift risk. If any copy has a different answer, extraction creates a coupling point that makes both cases harder to change independently. Occurrence count is a proxy for this question — at 3+ copies the change drivers are usually shared; at 2 copies they often diverge — but the change-driver answer takes precedence over the count when it is clear.
- Before "cleaning up" repetition, identify the change driver for each repeated block. If any block changes for a different reason, keep them separate even when the code looks similar today.
- If an existing shared abstraction has accumulated conditionals to handle special cases, compare the abstraction against inlining at each caller: count caller-specific branches, options used by only one caller, and behavior differences hidden behind the shared name.

See `examples/wrong-abstraction.md` for contrast cases where change-driver analysis decides whether similar code should stay separate or repeated logic should be consolidated.

## 2. Complexity Placement

**Question**: Which layer should own this complexity? Is the change pushing complexity toward the right boundary or scattering it?

**Key signals**:
- Complexity has two sources: **dependencies** (how much surrounding context a reader must load to understand this code) and **obscurity** (information that is hidden, unnamed, or implicit). Reducing these two matters more than any other optimization — more than performance, line count, or clever structure.
- Deep modules are better than shallow modules. A module with a simple interface that hides complex implementation is more valuable than one with a complex interface and trivial implementation.
- If callers need to know implementation details to use a function correctly, the abstraction boundary is in the wrong place. A self-explanatory interface means the name and signature alone communicate what the function does and what constraints apply — no source-reading required. Example: `cache.get_or_compute(key, fn, ttl)` is self-explanatory; `cache.get(key)` followed by manual set/expire logic is not.
- Pull complexity downward: if you can make 5 callers simpler by making 1 implementation more complex, do it. The net cognitive load decreases.
- Before adding a configuration parameter or flag, check whether the module already has the information needed to choose the behavior internally. If yes, keep the decision inside the module; each exported configuration parameter becomes caller-owned complexity.
- If validation, transformation, or error handling for one concern appears in multiple layers, identify the layer that has enough context to own the whole concern; scattered partial handling creates inconsistent behavior.

See `examples/complexity-misplacement.md` for a case where partial validation spread across layers creates a false sense of safety.

## 3. Failure Mode Awareness

**Question**: How will this code fail? Are the failure paths explicit and handled at the right level?

**Key signals**:
- Error handling is most effective at the ends (end-to-end principle). If every intermediate layer handles errors partially, you end up with duplicated, inconsistent recovery logic and errors that get swallowed or transformed beyond recognition.
- When adding error handling, ask: who is the right party to make the recovery decision? Often it is the outermost caller who has the full context, not the inner function that detected the problem.
- Classify each new or changed error path as retryable, user-facing, or programmer-error. If one handler treats multiple classes the same way, it can both swallow bugs and create noisy false alarms.
- When a function can fail in ways the caller doesn't check, the code has an implicit failure path. Make failure paths explicit, even if the handling is "propagate to the caller."
- For any operation with multiple state-changing steps, enumerate the persisted state after each possible failure point. If a failure leaves state that cannot be retried, rolled back, or surfaced to a recovery owner, the failure path is unresolved.

## 4. Deletability and Replaceability

**Question**: If requirements change, how easy is it to delete or replace this code? Is the change making the codebase more or less entangled?

**Key signals**:
- Code that is easy to delete is better than code that is easy to extend. Extension assumes you know the future; deletion works regardless.
- Isolate the parts that are likely to change from the parts that are likely to stay stable. Business rules change fast; data structures change slowly; protocols change slowest.
- If deleting this code would require changes in many unrelated modules, the coupling is too tight. Loose coupling means you can change your mind without rewriting the world.
- Feature flags, runtime configuration, and clear module boundaries all increase deletability. Deep inheritance hierarchies, global state, and implicit dependencies decrease it.
- When building something new and uncertain, favor a "big lump" that can be deleted as a whole over many small interlocking pieces that can only be deleted piecewise.

## 5. Boundary Contract and Dependency Direction

**Question**: What behavior does this boundary promise to callers, and which side of the boundary now knows about the other?

**Key signals**:
- Caller contract: identify behavior a caller can observe, including return values, thrown errors, defaults, side effects, ordering, persistence, emitted events, caching, sync/async timing, and reference identity. During a refactor, keep this behavior unchanged unless the task explicitly asks to change it.
- Treat observed behavior as the contract even when it was never documented. If a function has always returned an empty array on error, callers may rely on that; changing it to throw is a breaking change.
- A signature-compatible change can still break the contract when it changes the old default path, execution timing, side effects, error behavior, or object identity. Prefer tests that assert caller-observable behavior over tests that assert implementation shape.
- For public APIs and shared libraries, caller compatibility outranks internal cleanliness unless the task includes a migration or versioning plan.
- Dependency direction: if `A` imports, calls, constructs, references generated types from, reads configuration for, or otherwise must know about `B`, then `A` depends on `B`. This is a direction: `A -> B`.
- Prefer dependency edges from volatile or specific code toward stable or general code. Example: `features/billing` may depend on `lib/http`; `lib/http` depending on `features/billing/planRules` is wrong-way because every billing policy change can now force infrastructure changes.
- Treat a new import, package dependency, service call, generated type, callback, or configuration read across a module/package boundary as a stronger edge than a local dependency because it is harder to remove, test independently, version, and deploy.
- Before importing a helper "just for one function," inspect the helper's imports and package dependencies; the dependency includes that transitive surface, not only the symbol being called.
- Circular dependencies are a structural problem, not just a lint warning. They mean two modules cannot be understood, tested, or deployed independently.
- If two modules must now change together even without an import edge, surface the missing boundary decision: either keep the behavior local, move the shared primitive to a lower stable layer, or make the ownership dependency explicit.

See `examples/implicit-contract-break.md` for a case where a "safe" refactoring breaks callers through implicit behavioral changes.

## 6. Performance Awareness

**Question**: What is the performance impact of this change? Is the change respecting the performance boundaries of the system?

**Key signals**:
- Hot path vs cold path: Is this code on a hot path? Changes on hot paths deserve more scrutiny for performance.
- Algorithmic complexity: Does this change introduce O(n²) or worse complexity where O(n) or O(n log n) was expected? Does it add work inside a loop that could be done outside?
- I/O on the critical path: Does the change add database queries, network calls, or disk I/O to a path that was previously CPU-only? Every I/O call is a potential latency cliff.
- N+1 patterns: Does the change query a resource per item in a collection when a batch query would serve? This is the single most common performance regression in database-backed systems.
- Memory allocation patterns: Does the change create objects in a tight loop, build unbounded collections, or hold references that prevent garbage collection?
- Caching and memoization: If the same computation is performed repeatedly with the same inputs, compare the saved work against stale-data risk. Do not add caching unless invalidation, TTL, or data freshness expectations are visible in code or requirements.
- The premature optimization trap: Not every path needs optimization. If performance impact is speculative, measure before acting. But if the change is on a known hot path or introduces a known anti-pattern (N+1, unbounded collection, blocking I/O in async context), flag it — that is not premature, it is informed.

See `performance-dimensions.md` for deep guidance on each dimension: algorithmic fitness, I/O patterns (including latency reference tables), resource management, optimization judgment, frontend CWV, database query design, and async/concurrency bottlenecks.

## 7. Observability and Operability

**Question**: When this code fails in production, will the person debugging it at 3am have the information they need?

**Key signals**:
- Structured logging: Does the change add, preserve, or degrade logging at key decision points? Log entries should carry enough context (request ID, user ID, relevant identifiers) to reconstruct what happened without reproducing the issue.
- Error context propagation: When errors cross boundaries (function → caller → API → user), is enough context preserved? An error message that says "operation failed" helps nobody. An error that says "failed to update user 12345: database connection timeout after 30s" enables diagnosis.
- Metrics and alerting surface: If this change introduces a new failure mode, is there a way to detect it through metrics? A silent failure that accumulates over days is worse than a loud crash.
- Health checks and readiness signals: If the change affects system readiness (new dependency, new resource requirement), does the health check reflect this?
- Debuggability under pressure: Could someone unfamiliar with this code, reading only logs and metrics, understand what happened? The audience is a tired on-call engineer, not the original author.

## 8. Security Posture

**Question**: Does this change maintain or improve the system's security posture? What attack surface does it introduce?

**Key signals**:
- Input validation: Does the change handle untrusted input? Trust-boundary validation belongs at the boundary that receives external data: parse, normalize, size-limit, type-check, and reject unsafe shapes before the data enters the system. Domain validation belongs in the layer with business context. Do not scatter the same validation concern across both layers.
- Authentication and authorization: If the change exposes a new endpoint, route, or data path, are auth checks in place? Check that authorization is enforced at the right layer — not just at the UI, but at the API/service layer.
- Data exposure: Does the change risk exposing sensitive data in logs, error messages, API responses, or URLs? PII, credentials, tokens, and internal system details should never appear where they shouldn't.
- Dependency trust: If the change adds a new dependency, what is the trust level? Evaluate the dependency's maintenance status, security history, and the blast radius of a supply-chain compromise.
- Principle of least privilege: Does the change request only the permissions it needs? Over-privileged code is a larger blast radius when compromised.
- Cryptographic hygiene: If the change involves encryption, hashing, or token generation, are current algorithms and practices used? Hardcoded secrets, weak hashing (MD5/SHA1 for security), and custom crypto are red flags.

## 9. State Management and Data Integrity

**Question**: Does this change correctly manage shared state? Will state transitions remain consistent under all conditions — including concurrent access, partial failure, and restart?

**Key signals**:
- **Shared mutable state is the most common source of subtle bugs in refactoring.** When moving code between components, trace every piece of state the code reads or writes. If a refactor splits a function that previously managed state atomically into two functions that now manage it across a boundary, you have created a consistency window that didn't exist before.
- **In long-running processes, also ask whether state accumulates over time.** Persistent mutable state (deduplication sets, caches, counters, buffers) that grows with each unit of processed work and has no eviction or size bound will exhaust memory in proportion to total work processed. This is invisible in tests — it surfaces only under sustained production load. When reviewing state in an event consumer, server, or daemon, confirm that each piece of persistent mutable state either has a bounded key space by design or has an explicit eviction/rotation mechanism.
- **Identify the source of truth.** For every piece of state this code touches, there should be exactly one authoritative source. If data is denormalized or cached, the code must handle staleness explicitly — not assume freshness. When the same data exists in two places, ask which one wins on conflict and what happens when they disagree.
- **State transitions should be explicit and enforceable.** If an entity can be in states A, B, C, the valid transitions (A→B, B→C, but not A→C) should be enforced in code, not implied by call ordering. The question is not "does the current code path follow the right order?" but "what prevents a future caller from violating the order?"
- **Crash recovery determines real data integrity.** If the system fails mid-operation, what state is persisted? Any operation that modifies persistent state in multiple steps either needs atomicity (transaction) or idempotent recovery (can be safely re-run).
- **Local state and remote state will diverge.** When integrating external services, a successful API call followed by a local crash means the remote side has progressed but your system doesn't know. Design for this — idempotency keys, reconciliation jobs, or explicit "pending" states.
- **Schema migrations create a dual-state window.** Any migration that changes schema while the application is running creates a window where old code reads new schema (or vice versa). Ensure migrations are backward-compatible with the currently deployed code, or coordinate deployment order explicitly.

## 10. Migration and Backward Compatibility

**Question**: Does this change require a migration — of data, configuration, API consumers, or deployment sequence? Is the migration safe to run in production?

**Key signals**:
- Schema migrations that rename or remove columns while the application is running will break queries in the old code. Safe migrations add first, backfill, deploy new code, then remove old columns in a subsequent migration. A single migration that both adds and removes is a production risk.
- API changes that modify response shapes, remove fields, or change error formats break existing consumers. Even "internal" APIs may have consumers you don't know about — scripts, cron jobs, partner integrations. Treat all API changes as potentially breaking unless proven otherwise.
- Configuration changes that rename environment variables, change default values, or alter config file structure can break deployments silently. The application starts, but behaves differently in ways that won't surface until a specific code path is hit.
- Feature flags and gradual rollout reduce blast radius. When the change is risky or affects a large user base, identify whether rollback without a deploy exists; if not, surface the release risk.
- Deployment ordering matters. If this change requires "deploy service A before service B," document that explicitly. Implicit deployment ordering is a deployment outage waiting to happen.
