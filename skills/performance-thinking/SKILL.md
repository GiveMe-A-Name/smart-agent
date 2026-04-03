---
name: performance-thinking
description: "Invoke when the code being written or reviewed will run on a hot path, process significant data volumes, serve user-facing latency-sensitive requests, or when performance requirements exist. Cost of unnecessary invocation: a short performance review pass. Cost of missing: a performance problem discovered in production that is orders of magnitude more expensive to fix under load than to prevent during development. When uncertain whether performance matters for this change, invoke."
---

# Performance Thinking

Think about performance as a design constraint, not a post-hoc optimization. The most expensive performance problems are not slow algorithms — they are architectural decisions that cannot be fixed without redesign.

## Trigger Logic

**Invocation default**: Performance problems found in production under load are orders of magnitude more expensive to fix than performance-aware decisions made during development. The cost of an unnecessary performance review is a few minutes of analysis. The cost of missing it is a redesign under production pressure. Invoke when the change is on a path where performance might matter.

Use when:
- code runs on a hot path (request handling, event processing, data pipeline)
- code processes data volumes that grow with usage
- code serves latency-sensitive user-facing requests
- explicit performance requirements or SLAs exist
- code introduces new database queries, external API calls, or I/O operations

Do not use when:
- the code is a one-time script, migration, or admin tool where execution time is irrelevant
- the code is off the critical path, runs infrequently, and processes small data

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

- Never add optimization complexity without measurement proving it targets the actual bottleneck — except for structural anti-patterns (N+1, unbounded collections, missing timeouts) where cost/benefit is unambiguous.
- Optimize where time is actually spent, not where it is convenient. Amdahl's Law: optimizing 10% of the workload by 50% saves 5% total; optimizing 90% of the workload by 10% saves 9%.
- An optimization that makes code harder to understand requires profiling evidence. An optimization that improves both performance and clarity does not.
- Performance requirements must be stated and measurable before performance work begins. "Feels fast" is not a requirement.

## Decision Signals

Work through the dimensions relevant to the code at hand. See `references/performance-dimensions.md` for full depth on each.

**Path Criticality** — Is this code on a hot path?
- Request handlers, event processors, jobs that scale with data volume → hot.
- Admin tools, one-time migrations, infrequent jobs with small data → cold: note obvious anti-patterns only, skip optimization.

**Algorithmic Fitness** — Does the approach scale with production data?
- Nested loops over growing collections: O(n²) is fine at n=50, dangerous at n=50,000. Ask what n is in production.
- Repeated linear scans inside a loop → Set or Map lookup converts O(n) to O(1).

**I/O and Network** — How much I/O does this code do, and is it necessary?
- N+1: one query/call per item instead of batching — the single most common performance regression. Always fixable.
- Sequential independent calls: A then B then C when they don't depend on each other. Parallelize → latency = max(A, B, C).
- Missing timeout on external call → one slow dependency freezes the caller.

**Resource Management** — Are resources bounded and released?
- Unbounded collection loaded into memory → OOM at production scale.
- Connection/handle opened but not closed in error paths → pool exhaustion.
- Cache with no size bound or TTL → memory leak.

**Optimization Judgment** — Should I optimize this at all?
- No measurement, not on hot path, hypothetical future load, saves microseconds in an I/O-dominated path → do not optimize.
- Profiling shows bottleneck, or it's a well-known structural anti-pattern → optimize.
- Amdahl's Law applies: confirm you are optimizing the dominant component, not the convenient one.

**Frontend** — Does this create user-perceivable slowness?
- Bundle size matters only for the critical-path bundle blocking time-to-interactive.
- Profile re-renders before adding memoization — fix the structural cause, not the symptom.
- CWV (LCP < 2.5s, INP < 200ms, CLS < 0.1) are the metrics that matter.

**Database** — Are queries efficient at production scale?
- Verify index support on WHERE/JOIN columns before shipping a query on a hot path.
- Run EXPLAIN before assuming a query is efficient.

**Async and Concurrency** — Does concurrent code perform under load?
- CPU-intensive work on the event loop blocks all concurrent requests — offload to worker threads.
- Every queue must have a bound with explicit backpressure strategy.

## Failure Signals

Stop and reassess if:
- you are optimizing without profiling data, claiming "this should be faster," or adding complexity for a gain that hasn't been measured
- you are adding a database query or external API call inside a loop
- you are loading an unbounded dataset into memory
- you are optimizing a component that accounts for a small fraction of total latency (wrong bottleneck)
- you are looking at P50 latency while ignoring P99 tail behavior
- domain-specific anti-pattern present: memoization added without profiling re-renders, query shipped without checking EXPLAIN, CPU-intensive work running on the event loop, pool sized by intuition not workload

## Completion Criteria

- [ ] It was determined whether this code is on a performance-sensitive path.
- [ ] If on a hot path, the relevant dimensions (algorithmic fitness, I/O, resources) were checked.
- [ ] Structural anti-patterns (N+1, unbounded collections, missing timeouts) were checked regardless of path.
- [ ] Any recommended or applied optimization is backed by evidence that it targets the actual bottleneck.
- [ ] Domain-specific checks applied where relevant: index support for DB queries, bundle/CWV for frontend, event-loop blocking and backpressure for async.
- [ ] If no optimization was made, that decision rests on confirmed path characteristics — not skipped analysis.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether performance was genuinely addressed.

Am I treating the absence of immediate slowness as evidence that no performance work was needed?

Am I exiting because performance is genuinely addressed, or because the current code looks acceptable under light mental simulation?
