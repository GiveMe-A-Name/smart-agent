# Performance Dimensions — Detailed Reference

Deep guidance for each dimension. Use when a performance signal in SKILL.md requires deeper analysis.

---

## Navigating the Dimensions

| What you see in the code | Start with | Why |
|---|---|---|
| New code in a request handler, API endpoint, or event processor | Dimension 1 (Path Criticality) | Determines whether deeper analysis is warranted |
| A loop over a collection, nested iteration, or sort/search on user data | Dimension 2 (Algorithmic Fitness) | Complexity problems hide in code that works fine in tests |
| New database query, HTTP call, file read, or external service call | Dimension 3 (I/O and Network) | I/O dominates latency in most systems |
| Object pools, caches, connection setup, large allocations, unbounded collections | Dimension 4 (Resource Management) | Resource leaks cause production incidents, not slow responses |
| Someone proposing "this should be faster" | Dimension 5 (Optimization Judgment) | The most important question is whether to optimize at all |
| Frontend code: component rendering, bundle imports, image/asset loading | Dimension 6 (Frontend) | User-perceived performance is different from server-side performance |
| Database queries: complex joins, missing indexes, full table scans | Dimension 7 (Database) | Query design matters more than code optimization |
| Async/concurrent code: event loops, thread pools, worker queues | Dimension 8 (Async and Concurrency) | Concurrency bottlenecks are invisible in single-threaded tests |

---

## 1. Path Criticality

**Question**: Is this code on a hot path?

The single most important performance judgment. Code on a cold path (admin tools, one-time migrations, infrequent background jobs with small data) does not need performance analysis beyond the runtime red lines in `SKILL.md`. Code on a hot path deserves attention on all dimensions.

**Hot path signals**:
- Request-handling code: anything in the chain from receiving a user request to sending a response — middleware, controllers, serializers, database queries within a request
- Event/message processors: code that runs per-event in a stream or queue scales with throughput; one slow handler backs up the entire pipeline
- Background jobs that scale with data: a nightly job over 100 records is cold; the same job over 10 million records is hot — ask what n looks like in production

**Cold path signals**:
- Startup/initialization code: runs once, almost never worth optimizing unless startup time is an explicit requirement
- Developer tooling and scripts: cold unless they run in CI on every commit with strict latency constraints

**The judgment**: Cold path → note structural anti-patterns only, skip other dimensions. Hot path → continue to the relevant dimensions.

---

## 2. Algorithmic Fitness

**Question**: Does the approach scale with the data it will process?

Code that passes tests with 10 items may be unusable with production data volumes. The question is not "what is the theoretical complexity?" but "given the data this code will actually process, does the approach hold up?"

**Key signals**:
- Nested loops over the same growing collection: O(n²) is fine at n=50, dangerous at n=50,000 — ask: what is n in production, and how fast is it growing?
- Repeated linear scans: searching an unsorted list inside a loop — a Set or Map lookup converts O(n) per lookup to O(1)
- Sorting when you only need min/max or top-k: a full sort is O(n log n) when a single pass or heap gives O(n) or O(n log k)
- String concatenation in a loop: in many languages, each concatenation allocates a new string — use a builder or join

**Complexity scale reference**:

| Complexity | n=100 | n=10,000 | n=1,000,000 |
|---|---|---|---|
| O(1) | 1 | 1 | 1 |
| O(log n) | 7 | 13 | 20 |
| O(n) | 100 | 10,000 | 1,000,000 |
| O(n log n) | 664 | 132,877 | 19,931,569 |
| O(n²) | 10,000 | 100,000,000 | 1,000,000,000,000 |

**The judgment**: Is n small and bounded, or does it grow with users or time? If small and bounded, O(n²) is often fine. If it grows, it is not.

---

## 3. I/O and Network Awareness

**Question**: How much I/O does this code do, and is it necessary?

In most systems, I/O dominates latency. The biggest performance wins usually come from eliminating unnecessary I/O, not from optimizing computation.

**Key signals**:
- **N+1 pattern**: one query/call per item instead of a batch — first check whether batching is available and preserves the required semantics
  ```
  # N+1: one query per user
  for user in users:
      orders = get_orders(user.id)           # N queries

  # Batched: one query total
  orders = get_orders_for_users(user_ids)    # 1 query
  ```
  This pattern appears everywhere: ORM lazy loading, REST API calls per item, file reads per record.
- **Sequential independent calls**: calling services A, B, C in sequence when they don't depend on each other — total latency = A+B+C; parallelizing gives max(A,B,C)
- **Missing timeouts on external calls**: one slow dependency freezes the caller indefinitely — every external call needs a timeout
- **Redundant I/O**: fetching the same data multiple times within a single request because it's not passed between functions
- **Unbounded queries**: SELECT * FROM table with no LIMIT, WHERE, or pagination

**Latency orders of magnitude**:

| Operation | Approximate Latency |
|---|---|
| L1 cache reference | 1 ns |
| Main memory reference | 100 ns |
| SSD random read | 16 μs |
| Read 1 MB from SSD | 50 μs |
| Same-datacenter round trip | 500 μs |
| Read 1 MB from network | 10 ms |
| Cross-continent round trip | 150 ms |

Memory is ~100x faster than SSD; SSD is ~200x faster than a cross-datacenter network call. Adding a network call to a previously in-memory path adds 5–6 orders of magnitude of latency.

**The judgment**: When you see I/O on a hot path — is this call necessary? Can it be batched? Can independent calls be parallelized? Can the result be passed rather than re-fetched? These are almost always worth fixing without profiling.

---

## 4. Resource Management

**Question**: Does this code manage resources responsibly?

Resource problems don't show up as slow responses — they show up as production incidents. Memory leaks, connection exhaustion, and unbounded growth cause outages, not latency regressions.

**Key signals**:
- **Unbounded collections in memory**: loading all results into a list instead of streaming or paginating — code that works with 1,000 records OOMs with 1,000,000
- **Connections not returned**: database connections, HTTP clients, or file handles opened but not closed in error paths — resource pools drain and the system stops
- **Caches without eviction**: a cache that grows forever is a memory leak — every cache needs a size bound or TTL
- **Cache stampede**: when a popular cache key expires, all concurrent requests hit the backend simultaneously — use lock-based repopulation or staggered TTLs
- **Holding transactions during I/O**: a database transaction held open while waiting for a network call blocks the connection for the duration
- **Large allocations in loops**: creating large temporary objects on every iteration instead of reusing or allocating once outside the loop
- **SELECT \***: retrieving all columns when only a few are needed wastes bandwidth and memory, and prevents covering index optimizations

**The judgment**: Resource problems fail under sustained production load, not in tests. When you see resource acquisition (connections, memory, cache entries), look for the corresponding release and bound.

---

## 5. Optimization Judgment

**Question**: Should I optimize this, or leave it alone?

More performance problems are caused by premature complexity than by unoptimized code.

**Do NOT optimize when**:
- You have no measurement showing the code is slow
- The code is not on a hot path and runs infrequently
- The optimization makes the code significantly harder to understand
- The optimization saves microseconds in a path dominated by milliseconds of I/O
- You are optimizing for a hypothetical future load that may never arrive
- You are optimizing the component that is easy to optimize rather than the one that matters — Amdahl's Law: if 90% of time is in component A and 10% in B, optimizing B by 50% saves 5% total; optimizing A by 10% saves 9%

**DO optimize when**:
- Profiling shows this code is a significant contributor to latency or resource usage
- The code is on a hot path AND the fix removes a directly observable structural anti-pattern, such as replacing one query per item with one batch query over the same key set — safe even without profiling because the before/after cost model is visible in the code
- Performance requirements exist and are not being met
- The optimization improves both performance and readability

**Signals that you're optimizing wrong**:
- "This should be faster because..." without measurement data
- Optimizing code that profiling shows is not the bottleneck
- Adding complexity (caching layers, denormalization, custom data structures) for gains that haven't been measured
- Looking at averages or P50 while ignoring tail latency (P99)

---

## 6. Frontend Performance

**Question**: Does this frontend code create user-perceivable slowness or waste?

Frontend performance is measured in time-to-interactive, visual stability, and responsiveness — not CPU cycles or throughput.

**Key signals**:
- **Bundle size matters only on the critical path**: every kilobyte of JavaScript on the initial load must be downloaded, parsed, and executed before the page becomes interactive — code-split at route boundaries; audit new dependencies against the bundle budget for the critical-path bundle only
- **Rendering performance is structural, not memoization**: unnecessary re-renders are the primary performance problem in component-based frameworks; the fix is almost always structural (lifting state, splitting components, stabilizing references) not sprinkling `React.memo` or `useMemo` — profile first with framework devtools
- **Core Web Vitals are the metrics that matter**: LCP < 2.5s, INP < 200ms, CLS < 0.1 — if a change degrades CWV on affected pages, it needs attention regardless of code cleanliness
- **Images are often the largest payload**: for newly added images, use WebP/AVIF when supported, serve responsive sizes, and lazy-load below-the-fold images
- **Layout shifts destroy perceived quality**: always set explicit dimensions on images and embeds

---

## 7. Database Performance

**Question**: Are the database interactions designed for efficiency at production scale?

**Key signals**:
- **Index design determines query performance**: a query without a supporting index does a full table scan — for any query on a hot path: verify an index exists on WHERE/JOIN columns, composite indexes match query column order, and covering indexes are used for read-heavy queries
- **Understand the query execution plan**: EXPLAIN (or equivalent) before assuming a query is efficient — look for: sequential scans on large tables (missing index), nested loop joins on large datasets, sorts that spill to disk
- **Connection pool sizing is critical**: too few: requests queue waiting for a connection; too many: database overwhelmed with context-switching — monitor pool wait time; if it's non-zero, the pool is a bottleneck
- **Avoid SELECT \* on wide tables**: retrieve only needed columns; prevents the database from using covering indexes

---

## 8. Async and Concurrency Performance

**Question**: Does this async or concurrent code use resources efficiently without introducing bottlenecks or starvation?

**Key signals**:
- **Event loop blocking is the cardinal sin of async code**: in event-loop runtimes (Node.js, Python asyncio, browser JavaScript), any CPU-intensive synchronous operation blocks all other concurrent work — offload CPU-intensive work to worker threads or separate processes
- **Thread pool sizing is judgment, not formula**: CPU-bound work: threads ≈ CPU cores; I/O-bound work: threads can be much higher (tens to hundreds) because they spend most time waiting — an I/O-bound thread pool sized at CPU count wastes available concurrency
- **Backpressure prevents cascade failure**: when a producer creates work faster than a consumer can process it, unbounded queues grow until memory is exhausted — every queue must have a bound with explicit shedding strategy
- **Connection reuse over connection creation**: creating a new TCP/TLS connection per request adds 1–100ms overhead — use connection pools for databases, HTTP keep-alive for API calls
- **Async does not mean parallel**: in single-threaded async runtimes, `await Promise.all([a(), b(), c()])` runs concurrently only if A, B, C are I/O-bound; CPU-bound tasks run sequentially
