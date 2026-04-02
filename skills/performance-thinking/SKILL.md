---
name: performance-thinking
description: "Invoke when the code being written or reviewed will run on a hot path, process significant data volumes, serve user-facing latency-sensitive requests, or when performance requirements exist. Cost of unnecessary invocation: a short performance review pass. Cost of missing: a performance problem discovered in production that is orders of magnitude more expensive to fix under load than to prevent during development. When uncertain whether performance matters for this change, invoke."
---

# Performance Thinking

Think about performance as a design constraint, not a post-hoc optimization. The most expensive performance problems are not slow algorithms — they are architectural decisions that cannot be fixed without redesign.

This skill teaches the judgment to recognize when performance matters for the code you are looking at right now, what to do about it, and — critically — when to leave it alone.

## Trigger Logic

**Invocation default**: Performance problems found in production under load are orders of magnitude more expensive to fix than performance-aware decisions made during development. The cost of an unnecessary performance review is a few minutes of analysis. The cost of missing it is a redesign under production pressure. Invoke when the change is on a path where performance might matter.

Use when:
- code runs on a hot path (request handling, event processing, data pipeline)
- code processes data volumes that grow with usage (user data, logs, transactions)
- code serves latency-sensitive user-facing requests
- explicit performance requirements or SLAs exist
- code introduces new database queries, external API calls, or I/O operations

Do not use when:
- the code is a one-time script, migration, or admin tool where execution time is irrelevant
- the code is off the critical path and runs infrequently with small data
- performance has been measured and is within acceptable bounds

## Capability Boundary

This skill owns:
- judging whether a code change has performance implications worth addressing
- identifying which performance concerns matter for the code at hand
- deciding when optimization is warranted vs. premature
- recognizing common performance anti-patterns during writing or review

This skill does not own:
- benchmarking infrastructure or load testing tool selection
- capacity planning for production systems
- performance tuning of databases, networks, or operating systems
- choosing between fundamentally different architectures (that is a design decision)

## Invariants

- Measure before optimizing — intuition about performance is wrong more often than right.
- Optimize the bottleneck, not the thing that is easiest to optimize.
- Performance is a feature — it has requirements, it can regress, and it must be tested.
- Every optimization has a cost (complexity, readability, maintainability). The benefit must outweigh the cost.
- "Premature optimization is the root of all evil" does not mean "ignore performance until production is on fire." It means don't optimize without evidence.

## Completion Criteria

- [ ] It was identified whether this code is on a performance-sensitive path.
- [ ] If the code is on a hot path, the relevant dimensions (algorithmic fitness, I/O, resources) were checked.
- [ ] Structural anti-patterns that are always worth fixing (N+1, unbounded collections, missing timeouts) were checked.
- [ ] If frontend code, bundle size, render performance, and Core Web Vitals impact were considered.
- [ ] If database code, index support was verified and the query execution plan was checked.
- [ ] If async or concurrent code, event-loop blocking, pool sizing, and backpressure were considered.
- [ ] Any recommended or applied optimization is backed by evidence that it targets the actual bottleneck.
- [ ] If no optimization was made, that decision rests on confirmed path characteristics or hypothetical concern — not skipped analysis.
- [ ] Any applied optimization justifies its complexity cost.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether performance was genuinely addressed.

Am I treating the absence of immediate slowness as evidence that no performance work was needed?

Am I exiting because performance is genuinely addressed, or because the current code looks acceptable under light mental simulation?

---

## Reading the Situation

Before working through judgment dimensions, locate the code in this table to focus your analysis.

| What you see in the code | Start with | Why |
|---|---|---|
| New code in a request handler, API endpoint, or event processor | Dimension 1 (Path Criticality) | Determine if this is a hot path before analyzing anything else |
| A loop over a collection, nested iteration, or sort/search on user data | Dimension 2 (Algorithmic Fitness) | Complexity problems hide in code that works fine in tests |
| New database query, HTTP call, file read, or external service call | Dimension 3 (I/O and Network Awareness) | I/O dominates latency in most systems — one unnecessary call can dwarf all other costs |
| Object pools, caches, connection setup, large allocations, unbounded collections | Dimension 4 (Resource Management) | Resource leaks and unbounded growth cause production incidents, not slow responses |
| Someone (including you) saying "this should be faster" or "let's optimize this" | Dimension 5 (Optimization Judgment) | The most important question is whether to optimize at all |
| Code that is off the critical path, runs infrequently, or handles small data | Dimension 5 (Optimization Judgment) | Likely does not need optimization — confirm and move on |
| Frontend code: component rendering, bundle imports, image/asset loading | Dimension 6 (Frontend Performance) | User-perceived performance is different from server-side performance |
| Database queries: complex joins, missing indexes, full table scans, connection management | Dimension 7 (Database Performance) | Database is often the bottleneck — query design matters more than code optimization |
| Async/concurrent code: event loops, thread pools, worker queues, parallel processing | Dimension 8 (Async and Concurrency) | Concurrency bugs and bottlenecks are invisible in single-threaded tests |

---

## Judgment Dimensions

Work through the dimensions relevant to the code at hand. Not every dimension applies to every change — but skipping a relevant one is how performance problems ship.

### 1. Path Criticality

**Question**: Is this code on a hot path?

The single most important performance judgment is whether the code you're looking at is on a path where performance matters. Code on a cold path (admin tools, one-time migrations, infrequent background jobs processing small data) does not need performance analysis. Code on a hot path (request handling, event processing, anything that scales with user traffic) deserves careful attention.

**Key signals**:
- **Request-handling code**: anything in the chain from receiving a user request to sending a response is hot. Middleware, controllers, serializers, database queries within a request — all hot.
- **Event/message processors**: code that runs per-event in a stream or queue scales with throughput. One slow handler backs up the entire pipeline.
- **Background jobs that scale with data**: a nightly job over 100 records is cold. The same job over 10 million records is hot. Ask: what does N look like in production?
- **Startup/initialization code**: runs once — almost never worth optimizing unless startup time itself is a requirement.
- **Developer tooling and scripts**: cold unless they run in CI on every commit, in which case they are on a different hot path.

**The judgment**: If the code is not on a hot path, you can usually stop here. Note any obvious anti-patterns (unbounded queries, missing timeouts) but don't invest in optimization. If it is on a hot path, continue to the relevant dimensions below.

### 2. Algorithmic Fitness

**Question**: Does the approach scale with the data it will process?

Code that passes tests with 10 items may be unusable with production data volumes. The question is not "what is the theoretical complexity?" but "given the data this code will actually process, does the approach hold up?"

**Key signals**:
- **Nested loops over the same growing collection**: O(n²) is fine at n=50, dangerous at n=50,000. Ask: what is n in production, and how fast is it growing?
- **Repeated linear scans**: searching an unsorted list inside a loop. A Set or Map lookup converts O(n) per lookup to O(1).
- **Sorting when you only need min/max or top-k**: a full sort is O(n log n) when a single pass or heap gives you O(n) or O(n log k).
- **String concatenation in a loop**: in many languages, each concatenation allocates a new string. Use a builder or join.
- **Quadratic DOM or API response building**: constructing output by repeatedly appending to/searching within a growing structure.

**Reference — how complexity scales**:

| Complexity | n=100 | n=10,000 | n=1,000,000 |
|---|---|---|---|
| O(1) | 1 | 1 | 1 |
| O(log n) | 7 | 13 | 20 |
| O(n) | 100 | 10,000 | 1,000,000 |
| O(n log n) | 664 | 132,877 | 19,931,569 |
| O(n²) | 10,000 | 100,000,000 | 1,000,000,000,000 |

**The judgment**: The question is not "can I find a theoretically better algorithm?" It is "will the current approach break at realistic production data sizes?" If n stays small and bounded, O(n²) is fine. If n grows with users or time, it is not.

### 3. I/O and Network Awareness

**Question**: How much I/O does this code do, and is it necessary?

In most systems, I/O dominates latency. A 1ms in-memory computation is invisible next to a 50ms database query or a 150ms cross-service call. The biggest performance wins usually come from eliminating unnecessary I/O, not from optimizing computation.

**Key signals**:
- **N+1 pattern**: one query/call per item instead of a batch. This is the single most common performance regression in database-backed systems. The fix is always the same: batch the operation.
  ```
  # N+1: one query per user
  for user in users:
      orders = get_orders(user.id)    # N queries

  # Batched: one query total
  orders = get_orders_for_users(user_ids)  # 1 query
  ```
  This pattern appears everywhere: ORM lazy loading, REST API calls per item, file reads per record.
- **Sequential independent calls**: calling services A, B, C in sequence when they don't depend on each other. Total latency = A + B + C. Parallelizing gives latency = max(A, B, C).
- **Missing timeouts on external calls**: one slow dependency freezes the caller indefinitely. Every external call needs a timeout.
- **Redundant I/O**: fetching the same data multiple times within a single request because it's not passed between functions.
- **Unbounded queries**: `SELECT * FROM table` with no LIMIT, WHERE, or pagination. One bad query returns the entire table.

**Reference — latency orders of magnitude**:

| Operation | Approximate Latency |
|---|---|
| L1 cache reference | 1 ns |
| Main memory reference | 100 ns |
| SSD random read | 16 μs |
| Read 1 MB from SSD | 50 μs |
| Same-datacenter round trip | 500 μs |
| Read 1 MB from network | 10 ms |
| Cross-continent round trip | 150 ms |

The ratios matter: memory is ~100x faster than SSD, SSD is ~200x faster than a cross-datacenter network call. Adding a network call to a previously in-memory path adds 5–6 orders of magnitude of latency.

**The judgment**: When you see I/O on a hot path, ask: is this call necessary? Can it be batched? Can independent calls be parallelized? Can the result be passed rather than re-fetched? These are almost always worth fixing regardless of measurement — the cost/benefit is obvious.

### 4. Resource Management

**Question**: Does this code manage resources responsibly?

Resource problems don't show up as slow responses — they show up as production incidents. Memory leaks, connection exhaustion, and unbounded growth cause outages, not latency regressions.

**Key signals**:
- **Unbounded collections in memory**: loading all results into a list instead of streaming or paginating. Code that works with 1,000 records OOMs with 1,000,000.
- **Connections not returned**: database connections, HTTP clients, or file handles opened but not closed in error paths. Resource pools drain and the system stops.
- **Caches without eviction**: a cache that grows forever is a memory leak with extra steps. Every cache needs a size bound or TTL.
- **Cache stampede**: when a popular cache key expires, all concurrent requests hit the backend simultaneously. Use lock-based repopulation or staggered TTLs.
- **Holding transactions during I/O**: a database transaction held open while waiting for a network call blocks the connection for the duration. Other requests queue behind it.
- **Large allocations in loops**: creating large temporary objects on every iteration instead of reusing or allocating once outside the loop.
- **SELECT \***: retrieving all columns when only a few are needed wastes bandwidth and memory, and prevents covering index optimizations.

**The judgment**: Resource problems are harder to catch than latency problems because they don't fail in tests — they fail under sustained production load. When you see resource acquisition (connections, memory, cache entries), look for the corresponding release and bound. If you can't find them, that's the bug.

### 5. Optimization Judgment

**Question**: Should I optimize this, or leave it alone?

This is the core judgment this skill teaches. The instinct to optimize is strong — resist it until you have reason. More performance problems are caused by premature complexity than by unoptimized code.

**Do NOT optimize when**:
- You have no measurement showing the code is slow
- The code is not on a hot path and runs infrequently
- The optimization makes the code significantly harder to understand
- The optimization saves microseconds in a path dominated by milliseconds of I/O
- You are optimizing for a hypothetical future load that may never arrive
- You are optimizing the component that is easy to optimize rather than the one that matters (Amdahl's Law: if 90% of time is in component A and 10% in B, optimizing B by 50% saves 5% total; optimizing A by 10% saves 9%)

**DO optimize when**:
- Profiling shows this code is a significant contributor to latency or resource usage
- The code is on a hot path AND the fix is well-understood (e.g., batching N+1 queries) — these are safe even without profiling because the cost/benefit is unambiguous
- Performance requirements exist and are not being met
- The optimization is simple and improves both performance and readability (e.g., moving a query out of a loop — this is not an optimization, it's a bug fix)

**Key signals that you're optimizing wrong**:
- "This should be faster because..." without measurement data
- Optimizing code that profiling shows is not the bottleneck
- Adding complexity (caching layers, denormalization, custom data structures) for gains that haven't been measured
- Looking at averages or P50 while ignoring tail latency (P99)
- Choosing a "faster" approach that obscures what the code actually does
- Treating every allocation, every loop, every string operation as a performance problem

**The judgment**: The right default is to write clear, correct code and optimize only when evidence says you should. The exceptions are well-known structural anti-patterns (N+1 queries, unbounded collections, missing timeouts) where the cost is obvious and the fix improves both performance and clarity. Everything else requires measurement first.

### 6. Frontend Performance

**Question**: Does this frontend code create user-perceivable slowness or waste?

Frontend performance is user-perceived performance — it is measured in time-to-interactive, visual stability, and responsiveness to input, not in CPU cycles or throughput. The judgment challenge is distinguishing between optimizations that users will actually notice and micro-optimizations that add complexity without visible benefit.

**Key signals**:
- **Bundle size matters — but only when it's on the critical path.** Every kilobyte of JavaScript on the initial load must be downloaded, parsed, and executed before the page becomes interactive. A 2MB bundle on a 3G connection takes 10+ seconds. But: a lazily-loaded module that only appears after user interaction has minimal impact on initial load. The judgment is not "is this dependency large?" but "does this dependency block time-to-interactive?" Code-split at route boundaries. Tree-shake unused exports. Audit new dependencies against the bundle budget — but only for the critical path bundle.
- **Rendering performance is about eliminating unnecessary work, not adding memoization everywhere.** In component-based frameworks (React, Vue, Svelte), unnecessary re-renders are the primary performance problem. But the fix is almost always structural — lifting state, splitting components, stabilizing references — not sprinkling `React.memo` or `useMemo` everywhere. Memoization without profiling data is premature optimization that adds cognitive load. Profile first with framework devtools. If re-renders are the bottleneck, fix the cause (unstable references, state too high in the tree), not the symptom.
- **Core Web Vitals are the metrics that matter for user experience.** Largest Contentful Paint (LCP < 2.5s), Interaction to Next Paint (INP < 200ms), Cumulative Layout Shift (CLS < 0.1). These measure what users actually experience. The judgment: if a change degrades CWV on the affected pages, it needs attention regardless of how clean the code is. If CWV are healthy, the performance is likely acceptable even if a synthetic benchmark shows room for improvement. Optimize for what users feel, not for lighthouse scores on a fast machine.
- **Images are often the largest payload — and the easiest win.** Use appropriate formats (WebP/AVIF over PNG/JPEG where supported), serve responsive sizes (not a 4000px image for a 400px container), and lazy-load below-the-fold images. An unoptimized hero image can add 5+ seconds to LCP. But: over-optimizing non-critical images (thumbnails, background decorations) yields diminishing returns. Focus on the LCP element first.
- **Layout shifts destroy perceived quality — but not all shifts are equal.** Elements jumping around as the page loads (ads loading, images without dimensions, fonts swapping) erodes user trust. Always set explicit dimensions on images and embeds, use font-display strategies, and reserve space for async content. The judgment: a CLS of 0.05 from a minor font swap is rarely worth engineering effort; a CLS of 0.3 from an ad injection that pushes content down is a real problem. Prioritize shifts that move content the user is actively reading.

### 7. Database Performance

**Question**: Are the database interactions in this code designed for efficiency at production scale?

Database queries are often the dominant source of latency. A single unoptimized query can make an otherwise fast system unusable.

**Key signals**:
- **Index design determines query performance.** A query without a supporting index does a full table scan — O(n) instead of O(log n). For any query on a hot path: verify an index exists on the WHERE/JOIN columns, composite indexes match the query's column order, and covering indexes are used for read-heavy queries to avoid table lookups.
- **Understand the query execution plan.** EXPLAIN (or equivalent) is the database's answer to "how will you execute this?" Read it before assuming a query is efficient. Look for: sequential scans on large tables (missing index), nested loop joins on large datasets (may need hash join), sorts that spill to disk (missing index or insufficient work_mem).
- **Connection pool sizing is critical.** Too few connections: requests queue waiting for a connection. Too many: the database is overwhelmed with context-switching. The right size depends on the database (PostgreSQL recommends connections = (2 * CPU cores) + disk spindles for the DB server) and the application's query patterns. Monitor pool wait time — if it's non-zero, the pool is a bottleneck.
- **Read/write splitting for read-heavy workloads.** If the workload is 90% reads, sending all queries to the primary database wastes its capacity. Route read queries to replicas. Be aware of replication lag — recently written data may not be immediately available on replicas. For read-after-write consistency, route the reading query to the primary.
- **Avoid SELECT * on wide tables.** Retrieve only the columns you need. Wide rows waste bandwidth and memory, and prevent the database from using covering indexes. This matters most for tables with large text/blob columns or many columns.

### 8. Async and Concurrency Performance

**Question**: Does this async or concurrent code use resources efficiently without introducing bottlenecks or starvation?

**Perspective**: This dimension addresses the **performance characteristics** of concurrent code — throughput, resource utilization, and bottleneck identification. Whether async code is *correctly tested* (synchronization primitives, deterministic assertions, flaky test management) is a TDD concern, not a performance concern. Here the question is: given that the code works correctly, does it perform well under concurrent load?

Concurrency issues are among the hardest performance problems because they are invisible in single-threaded tests and intermittent in production.

**Key signals**:
- **Event loop blocking is the cardinal sin of async code.** In event-loop-based runtimes (Node.js, Python asyncio, browser JavaScript), any CPU-intensive synchronous operation blocks all other concurrent work. A 100ms JSON parse blocks 100ms of request processing for every other concurrent request. Offload CPU-intensive work to worker threads or separate processes.
- **Thread pool sizing is judgment, not formula.** For CPU-bound work: threads ≈ CPU cores. For I/O-bound work: threads can be much higher (tens to hundreds) because they spend most time waiting. An I/O-bound thread pool sized at CPU count wastes available concurrency; a CPU-bound pool sized at 100 creates context-switching overhead.
- **Backpressure prevents cascade failure.** When a producer creates work faster than a consumer can process it, unbounded queues grow until memory is exhausted. Every queue must have a bound. When the bound is reached, the system must either: reject new work (shed load), slow the producer (flow control), or drop low-priority work (priority-based shedding). "Accept everything and hope" is not a strategy.
- **Connection reuse over connection creation.** Creating a new TCP/TLS connection per request adds 1-100ms of overhead per request. Use connection pools for databases, HTTP keep-alive for API calls, and persistent connections for message queues. Monitor connection creation rate — high rates indicate missing pooling.
- **Async does not mean parallel.** In single-threaded async runtimes, concurrency means interleaving, not simultaneous execution. `await Promise.all([a(), b(), c()])` runs A, B, C concurrently only if they are I/O-bound. If they are CPU-bound, they run sequentially. Understanding this distinction prevents false expectations about async performance gains.

---

## Failure Signals

Stop and reassess if:
- You are optimizing code without profiling data showing it's a bottleneck
- You are adding a database query or external API call inside a loop
- You are loading an unbounded dataset into memory
- You are adding complexity for a performance gain that hasn't been measured
- You are optimizing the wrong component (spending time on the 10%, not the 90%)
- You are making performance claims without measurement ("this should be faster because...")
- You are ignoring tail latency (P99) while only looking at averages or P50
- Frontend: you are adding memoization everywhere "just in case" without profiling re-render waste
- Database: you are writing queries without checking the execution plan on production-scale data
- Async: you are running CPU-intensive work on the event loop / main thread
- You are sizing a thread/connection pool based on intuition rather than workload characteristics

