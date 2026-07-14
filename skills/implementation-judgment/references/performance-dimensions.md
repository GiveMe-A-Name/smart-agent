# Runtime and Performance Judgment

Use this reference when the proposed implementation adds I/O, work that scales with data, persistent resources, caching, database access, or concurrency. Evaluate the triggered signal only; performance analysis is not a default design phase.

## Establish the cost model

Before changing the design for performance, identify:

- how often the path runs;
- what input or state size controls its work;
- which operation dominates latency or resource use; and
- whether production evidence, a stated requirement, or a visible structural anti-pattern motivates the change.

Cold startup code, bounded admin tools, and small developer scripts rarely justify added complexity. Request paths, per-event handlers, and jobs whose work grows with production data deserve closer scrutiny.

## Growing work

Act when the implementation introduces an observable scaling hazard:

- an I/O call per item where one batch operation can preserve semantics;
- repeated scans or nested loops over an unbounded collection;
- loading an unbounded result set instead of streaming or pagination;
- sorting all data when only a bounded subset is needed; or
- repeated fetching of data already available in the current operation.

Do not optimize from complexity notation alone. Record whether the relevant size is small and bounded or grows with users, traffic, or time.

## External I/O and resources

For a new network, database, subprocess, or file dependency:

- follow the codebase's timeout and cancellation policy;
- confirm acquisition has release behavior on success and failure;
- avoid holding a transaction or scarce resource while waiting on unrelated I/O;
- bound collections, queues, caches, buffers, and per-key state that can grow over a long-running process; and
- prefer connection or client reuse when the existing platform provides it.

A cache is a state design, not a free optimization. Do not add one without a named freshness contract, invalidation or expiry behavior, and a size bound.

## Database access

When a hot path adds or changes a query, inspect the query shape and production-scale access pattern. Check for per-row queries, unbounded scans, unnecessary columns, and whether an existing index supports the actual filter, join, and ordering. Use an execution plan or measurement when index choice or query cost determines the design; do not infer performance from query text alone.

## Concurrency

Parallelize independent I/O only after confirming:

- operations have no required ordering or conflicting side effects;
- the dependency and local resource pools can tolerate the concurrency;
- failures and cancellation have defined semantics; and
- concurrency is bounded when the number of operations grows with input.

Async syntax does not make CPU work parallel. Avoid blocking an event loop with CPU-heavy or synchronous I/O, and use the runtime's established worker mechanism when offloading is necessary.

## Frontend changes

For user-facing paths, prefer measured user impact over generic micro-optimization. Inspect newly added critical-path JavaScript, large assets, avoidable network requests, render loops, and layout instability. Use the repository's bundle budgets, performance tests, or field metrics when available; do not introduce memoization, caching, or code splitting solely because a component looks complex.

## Stop condition

Choose the simplest design that satisfies the observed scale or requirement after the triggered hazard is resolved. Do not add speculative caching, batching infrastructure, concurrency, denormalization, or custom data structures for an unmeasured future load. If performance requirements remain uncertain and no structural hazard is visible, preserve the clear design and identify measurement as the next evidence gap.
