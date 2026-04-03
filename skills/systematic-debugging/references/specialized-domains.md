# Specialized Debugging Domains

The mental models in SKILL.md are universal. These domains add domain-specific heuristics for categories of bugs that require specialized thinking. Reach for this file when the judgment table in SKILL.md routes you to a specialized domain.

---

## Distributed Systems Debugging

Distributed systems fail in ways that monolithic systems cannot — and the failures are often partial, intermittent, and non-reproducible.

**Key heuristics:**
- **Assume the network is lying.** Timeouts, retries, and partial failures mean the same request can succeed on one node and fail on another. A "successful" operation may have been retried and executed twice.
- **Clocks are approximate.** Log timestamps across services are not perfectly synchronized. When correlating events across services, allow for clock skew (milliseconds to seconds). Use logical clocks or trace IDs, not wall-clock timestamps, to establish ordering.
- **Trace the request, not the component.** In a multi-service system, start with the distributed trace (trace ID) and follow the request through each service boundary. The bug often lives at a boundary — serialization, deserialization, protocol translation, or timeout configuration.
- **Check for partial failures.** If an operation involves steps A, B, C across services, and B fails: is A rolled back? Is C skipped? Distributed transactions are hard — most systems use eventual consistency, and the "eventually" part is where bugs hide.
- **Reproduce with fault injection.** If a bug only occurs under network partition, latency spike, or service degradation, use chaos engineering techniques (delay injection, connection dropping) to reproduce reliably.

---

## Performance Debugging

Performance bugs are different from correctness bugs — the system produces the right answer, just too slowly. The debugging approach must be measurement-driven, not hypothesis-driven.

**Key heuristics:**
- **Measure first, theorize second.** Profile before changing code. Use CPU profilers (flame graphs), memory profilers (allocation tracking), and I/O profilers (query logs, network traces). The bottleneck is rarely where you think it is.
- **Understand the bottleneck type.** A system can be CPU-bound, memory-bound, I/O-bound, or lock-contention-bound. The fix is completely different for each. Adding concurrency to a CPU-bound problem just adds overhead.
- **Look for the multiplier.** Most performance problems are not "this one thing is slow" but "this fast thing happens too many times." An operation that takes 1ms but runs 10,000 times per request is a 10-second problem. Find what is multiplying.
- **Amdahl's law applies.** If 90% of time is in function A and 10% in function B, optimizing B by 50% saves 5% total. Optimizing A by 10% saves 9%. Always optimize the dominant component.
- **Check for resource exhaustion.** Connection pool limits, thread pool starvation, file descriptor limits, memory pressure triggering GC — these create performance cliffs where the system works fine until it doesn't.
- **Tail latency matters.** P50 being fine while P99 is terrible often points to garbage collection pauses, lock contention, noisy neighbors, or cold cache scenarios. Don't just measure averages.

---

## Concurrency Debugging

Concurrency bugs are the hardest category — they are non-deterministic, rarely reproducible, and the act of observing them (adding logging) can change their behavior (Heisenbug).

**Key heuristics:**
- **Race conditions hide behind "usually works."** If something works 99% of the time but fails 1%, suspect a race condition. The question is: what shared state is being accessed without proper synchronization?
- **Deadlocks have a signature.** Two or more threads/processes waiting for each other, each holding a resource the other needs. Check for lock ordering violations. If code acquires lock A then lock B in one path, and lock B then lock A in another, deadlock is inevitable under load.
- **Ordering assumptions are implicit contracts.** "This always runs before that" is an assumption, not a guarantee — unless enforced by synchronization primitives. When debugging async code, map out every ordering assumption and check which ones are actually enforced.
- **Reproduce with stress testing.** Concurrency bugs surface under load. Increase thread count, add artificial delays at suspected race points, run the test thousands of times. Tools like ThreadSanitizer (TSan), Helgrind, or Go's race detector can find data races mechanically.
- **Immutability eliminates classes of bugs.** If data doesn't change after creation, it can't have a race condition. When you find a concurrency bug, consider whether making the data immutable is a better fix than adding locks.

---

## Data and State Debugging

When the system produces wrong results but doesn't crash, the problem is usually in the data — wrong state, stale state, or inconsistent state.

**Key heuristics:**
- **Find the first wrong value.** Trace backward from the wrong output to find where the data first became incorrect. Was it wrong in the database? Wrong after transformation? Wrong at input? The first wrong value points to the owning layer.
- **Check for stale data.** Caches, replicas, materialized views, and denormalized copies all go stale. If the "source of truth" is correct but the system behaves wrong, an intermediate copy is stale.
- **Schema drift.** When code and database schema evolve independently (especially during migrations), fields can be misinterpreted, default values can change meaning, and null handling can break.
- **Time-dependent data.** Data that was correct when written may be incorrect now. Timezone handling, expiration logic, and relative-time calculations are common sources of subtle data bugs.

---

## Build, Dependency, and Tooling Debugging

Build failures and dependency issues are a distinct category — the system doesn't even run, and the error messages often point to tooling internals rather than your code.

**Key heuristics:**
- **Read the full error, not just the last line.** Build tools often produce cascading errors. The real cause is usually the first error, not the last. Scroll up.
- **Distinguish between your code errors and toolchain errors.** A type error in your code is fundamentally different from a compilation error in a dependency or a misconfigured build tool. The fix path is completely different for each.
- **Version conflicts are combinatorial.** When dependency A requires library X v2 and dependency B requires library X v3, the resolution depends on the package manager's strategy. Understand whether your package manager deduplicates, hoists, or isolates. Check lock files — the resolved version may not be what the version range suggests.
- **"Works on my machine"** almost always means environment difference. Systematically diff: OS version, runtime version, package manager version, installed global tools, environment variables, and lock file state.
- **Stale artifacts cause phantom bugs.** Cached build outputs, stale compiled files, outdated lock files, and docker layer caches all cause situations where the source code is correct but the running artifact is wrong. When debugging makes no sense, clean all caches and rebuild from scratch before investing more time.
- **Monorepo-specific**: if a build fails after a seemingly unrelated change in another package, check for shared dependencies, build order, and workspace hoisting. The coupling is real even if the packages look independent.

---

## Configuration and Environment Debugging

When the code is correct but the system behaves wrong, the problem often lives outside the code — in configuration, environment variables, infrastructure, or external service state.

**Key heuristics:**
- **Print the actual config.** Don't assume the config file you're reading is the one the application loaded. Log the resolved configuration at startup. Environment variables may override file config. Multiple config sources may merge in unexpected order.
- **Environment variable precedence.** `.env` files, shell exports, Docker Compose env, Kubernetes ConfigMaps, CI/CD variables — these stack and override in platform-specific order. If a variable has the wrong value, trace the entire precedence chain.
- **Feature flags and A/B tests.** If behavior differs between environments or users, check whether a feature flag or experiment is active. These are invisible to code reading — the same code path executes different behavior based on runtime configuration.
- **External service state.** If your system integrates with external services, the external service may have changed (API version, rate limits, authentication requirements, data format) without your code changing. Check external service changelogs and status pages before blaming your code.
- **DNS, SSL, and network.** "Connection refused" and "timeout" often point to infrastructure (DNS resolution, certificate expiry, firewall rules, proxy configuration) rather than code. Check these before debugging application-level networking code.
