---
name: systematic-debugging
description: "Invoke when something is broken or behaving unexpectedly and the fix is not yet confirmed. Cost of unnecessary invocation: minor overhead. Cost of missing: a fix that masks the root cause — the bug returns in a different form. When in doubt, invoke before touching any code."
---

# Systematic Debugging

Debugging is an inverse problem: you observe effects and must infer causes. This skill equips you with the mental models, principles, and judgment to find root causes — not a fixed procedure to follow step by step. Assess the situation, choose the right thinking tool, and adapt as evidence emerges.

## Trigger Logic

**Invocation default**: The cost of unnecessary debugging is minor overhead. The cost of skipping it is a fix that doesn't hold or hides the real problem. Invoke before changing any code in response to a failure or unexpected behavior.

Use when:
- something is broken or behaving unexpectedly and the fix is not yet known
- a bug was reported but the root cause has not been established
- proceeding without investigation would require guessing what to change

Do not use when:
- the work is a feature or refactor with no current failure

## Capability Boundary

This skill owns the full debugging cycle: from a visible failure to a single verified fix. It does NOT own architectural changes — 3+ fix failures that keep exposing new coupling is an escalation signal, not a scope expansion.

## The Invariant

```
NO FIXES WITHOUT ROOT CAUSE FIRST
```

Root cause means you can state: *"The bug is at [specific location] because [mechanism], which produces [symptom]."*

The test: can you predict what will happen if you make a specific change? If yes, you have a root cause. If no, you have a hypothesis — keep investigating.


## Completion Criteria

- [ ] Root cause was established with evidence, not just a hypothesis.
- [ ] The bug can be stated as: "The bug is at [location] because [mechanism], which produces [symptom]."
- [ ] The fix outcome could be predicted before running it.
- [ ] The fix was verified against the original failure.
- [ ] Related bugs following the same pattern were checked.
- [ ] Environment, configuration, and build issues were ruled out before focusing on code logic.
- [ ] If significant time passed without progress, escalation happened with context instead of continued spinning.

**If any criterion is not met, return to the relevant mental model before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the bug is genuinely fixed.

Am I treating a symptom disappearance as proof of root-cause resolution?

Am I exiting because the bug is genuinely fixed, or because the latest run happened to look good enough?

---

## Mental Models

These are your thinking tools. Each one is a lens for seeing a bug differently. You don't use them in order — you reach for whichever one fits the situation.

### Debugging Is a Search Problem

A bug is one broken thing among thousands of working things. Every action you take should eliminate possibilities. If an action doesn't shrink the search space, it's wasted.

The expert debugger asks: *"What's the fastest experiment I can run that eliminates the most remaining possibilities?"* — not *"What might be wrong?"*

When the search space is large, use **divide and conquer**: split it in half, test which half contains the bug, discard the other, repeat. This gives logarithmic convergence — even in a million-line codebase, ~20 well-chosen binary splits find the bug.

Ways to split:
- **Through code**: checkpoint halfway through execution. Data correct at midpoint? Bug is in the second half.
- **Through time**: `git bisect`. "Worked Tuesday, broken now" → bisect finds the exact breaking commit.
- **Through components**: in layered systems (UI → API → Service → DB), check data at each boundary. Bug lives between the last correct boundary and the first incorrect one.
- **Through input**: some inputs trigger it, others don't. Bisect the input space — which field, which value range?

### Observation Over Theory

*"Quit thinking and look."* — Agans

The natural impulse is to theorize immediately. Resist it. Observation first, theory second. Theories formed without data are usually wrong — and worse, they anchor you to a false explanation.

What observation looks like in practice:
- Read the **full** error message. Not just the first line — the type, the full stack trace, the first frame in *your* code, the data values. "What the error says" and "where in your code it originated" are different questions.
- **Print the actual value.** Don't assume you know what a variable is. The thing you're most sure about ("that can't be null here") is often exactly what's wrong.
- **Instrument at boundaries.** In multi-component systems, add logging at each boundary, run once, read where data breaks. One instrumented run costs less than three wrong hypotheses.
- **Find a working example** of the same pattern and diff it against the broken case. List every difference, however small.

### Symptom Site ≠ Owning Layer

The file or line where the error surfaces is where the symptom appears, not necessarily where the fix belongs. A stack trace points to the symptom site; the owning layer requires tracing the behavior back to its source.

When you see an error at `file.ts:42`, your first question is NOT "what's wrong at line 42?" — it's *"what put the system in a state where line 42 fails?"* The answer often lives in a completely different file, layer, or even repository.

Before accepting the stack-trace location as the bug:
- **Expectation wrong?** Is the assertion itself testing for the wrong thing?
- **Fixture wrong?** Is the test setup producing bad input or state?
- **Environment wrong?** Is something outside the code (config, timing, deps) causing this?
- **Implementation wrong?** Is the production code actually at fault?

Check all four before committing to one.

### Causal Chain Depth

Every surface cause has a deeper cause. The depth at which you fix determines whether the bug class recurs or just this one instance.

Use the **5 Whys** to trace the chain:

1. Why did the API return 500? → Database query threw an exception.
2. Why did the query throw? → It received a null parameter.
3. Why was the parameter null? → The upstream service omitted it.
4. Why did upstream omit it? → The field was renamed in a migration but not in the serializer.
5. Why wasn't the serializer updated? → Migration and serializer are in different repos with no shared contract.

The fix at level 1 (null check) is a band-aid. The fix at level 5 (shared contract + test) eliminates the entire class of bug. Every time you think you've found the cause, ask one more "why."

### Reproduction as Understanding

If you can't make the bug happen on demand, you don't understand it yet.

- **Consistent reproduction** means you've identified the triggering conditions.
- **Minimizing the reproduction** reveals what the bug depends on. If a 1000-line test triggers it, can a 10-line test? Each piece you remove and the bug survives tells you what the bug does NOT depend on. A truly small reproduction often makes the cause obvious.
- **Fast reproduction** enables rapid iteration. If each attempt takes 3 minutes, invest time upfront to make it take 10 seconds. This pays for itself within a few cycles.
- **Intermittent failures** mean there's an uncontrolled variable — timing, ordering, external state. That variable IS the bug. Find it.

---

## Principles

These always hold, regardless of which mental model you're using.

1. **Evidence before action.** No editing until you can restate the failure in specific terms and point to evidence for which layer owns it.
2. **One variable at a time.** Change one thing per experiment. If you change three things and it works, you don't know which fixed it — and the other two may have introduced new bugs.
3. **Keep an audit trail.** Write down what you tried, in what order, what happened. When you're 2 hours in, you won't remember minute 15. Any detail could be the important one.
4. **Fix at the source.** The root cause tells you where to fix. If you fix at the symptom site, you're papering over the problem.
5. **Verify with the failing test.** Not "I think it should work" — run it. If there's no test, write one.
6. **If you didn't fix it, it ain't fixed.** If the bug disappeared and you don't know why, it will come back. Understand what changed.
7. **Check the plug.** Before diving deep, rule out the boring causes: stale build, wrong branch, wrong file, bad config, wrong environment, cached artifact. These account for a surprising percentage of "bugs."

---

## Judgment: Reading the Situation

This is the hard part — knowing which tool to reach for. Here's how experienced engineers read the situation:

| What you're seeing | What it suggests | Reach for |
|---|---|---|
| Clear error message pointing to a specific location | May be a direct hit, but verify — could be symptom site, not owner | **Symptom site ≠ Owning layer** — trace backward before editing |
| "It used to work" / something changed recently | Likely a regression | `git log`, `git bisect` — **divide and conquer through time** |
| Fails in full suite, passes in isolation | Test pollution or ordering dependency | `find-polluter.sh`, bisection — **reproduction** |
| Intermittent / flaky | Uncontrolled variable: timing, state, ordering | **Reproduction** — find the variable, see `condition-based-waiting.md` |
| Multi-component system, unclear which layer | Need to isolate the broken layer | **Observation** — instrument boundaries, check data at each crossing |
| You've formed a theory but haven't tested it | Risk of confirmation bias | **Search problem** — design an experiment that could DISPROVE your theory |
| 2+ failed hypotheses, feeling stuck | Assumptions may be wrong | **Check the plug**, question assumptions, get a fresh view — explain the problem aloud |
| Each fix exposes a new failure elsewhere | Structural/design problem, not a single bug | **Architecture escalation** — surface to human partner |
| Error deep in a call stack, unclear origin | Need to trace backward to the original trigger | **Causal chain depth** + `root-cause-tracing.md` |
| Fix worked but you want it to stay fixed | Need to prevent the bug class, not just this instance | **Causal chain depth** + `defense-in-depth.md` |
| Build fails after dependency update | Version conflict or breaking change in dep | **Build debugging** — read changelog between versions, check lock file diff |
| "Works on my machine" but fails in CI/staging | Environment difference | **Config/environment debugging** — diff runtime versions, env vars, config sources |
| Code looks correct but behavior is wrong | Configuration, feature flag, or stale artifact | **Config debugging** — print actual resolved config, clean caches, check feature flags |
| Error message references internal tooling, not your code | Build tool, bundler, or compiler issue | **Build debugging** — read the FIRST error (not last), check toolchain versions |
| Same code behaves differently for different users | Feature flag, A/B test, permissions, or data-dependent | **Data/state debugging** — check user-specific state, flags, permissions |

**The meta-skill**: after each action, ask *"Did this shrink my search space?"* If not, change approach. Don't repeat the same type of action hoping for a different result.

---

## Cognitive Traps

These are what actually make debugging hard. Technical complexity is secondary — psychology is primary.

| Trap | How it manifests | Countermeasure |
|---|---|---|
| **Confirmation bias** | You see evidence for your theory, ignore evidence against it | Actively try to DISPROVE your hypothesis, not prove it |
| **Anchoring** | You fixate on your first theory and bend all evidence to fit | After 2 failed experiments on the same theory, force yourself to generate 3 alternative explanations |
| **Proximity bias** | You blame the nearest code to the stack trace | Explicitly ask: "Is the symptom site the owning layer? What evidence do I have?" |
| **Sunk cost** | You keep a failing approach because you already invested hours | Time spent is gone. Ask: "What's the fastest path from HERE?" not "How do I salvage what I've done?" |
| **Availability bias** | You assume it's the same kind of bug you saw last week | Similar symptoms can have completely different causes. Check the evidence for THIS bug |
| **Assumption blindness** | You're certain something is true without verifying | The thing you're most confident about is often wrong. Print it. Check it. Verify. |
| **Theory-before-data** | You jump to "I bet it's X" before observing | Force yourself to state 3 observations before forming any hypothesis |

---

## Specialized Debugging Domains

The mental models above are universal. These domains add domain-specific heuristics for categories of bugs that require specialized thinking.

### Distributed Systems Debugging

Distributed systems fail in ways that monolithic systems cannot — and the failures are often partial, intermittent, and non-reproducible.

**Key heuristics:**
- **Assume the network is lying.** Timeouts, retries, and partial failures mean the same request can succeed on one node and fail on another. A "successful" operation may have been retried and executed twice.
- **Clocks are approximate.** Log timestamps across services are not perfectly synchronized. When correlating events across services, allow for clock skew (milliseconds to seconds). Use logical clocks or trace IDs, not wall-clock timestamps, to establish ordering.
- **Trace the request, not the component.** In a multi-service system, start with the distributed trace (trace ID) and follow the request through each service boundary. The bug often lives at a boundary — serialization, deserialization, protocol translation, or timeout configuration.
- **Check for partial failures.** If an operation involves steps A, B, C across services, and B fails: is A rolled back? Is C skipped? Distributed transactions are hard — most systems use eventual consistency, and the "eventually" part is where bugs hide.
- **Reproduce with fault injection.** If a bug only occurs under network partition, latency spike, or service degradation, use chaos engineering techniques (delay injection, connection dropping) to reproduce reliably.

### Performance Debugging

Performance bugs are different from correctness bugs — the system produces the right answer, just too slowly. The debugging approach must be measurement-driven, not hypothesis-driven.

**Key heuristics:**
- **Measure first, theorize second.** Profile before changing code. Use CPU profilers (flame graphs), memory profilers (allocation tracking), and I/O profilers (query logs, network traces). The bottleneck is rarely where you think it is.
- **Understand the bottleneck type.** A system can be CPU-bound, memory-bound, I/O-bound, or lock-contention-bound. The fix is completely different for each. Adding concurrency to a CPU-bound problem just adds overhead. Caching an I/O-bound problem helps; caching a CPU-bound problem doesn't.
- **Look for the multiplier.** Most performance problems are not "this one thing is slow" but "this fast thing happens too many times." An operation that takes 1ms but runs 10,000 times per request is a 10-second problem. Find what is multiplying.
- **Amdahl's law applies.** If 90% of time is in function A and 10% in function B, optimizing B by 50% saves 5% total. Optimizing A by 10% saves 9%. Always optimize the dominant component.
- **Check for resource exhaustion.** Connection pool limits, thread pool starvation, file descriptor limits, memory pressure triggering GC — these create performance cliffs where the system works fine until it doesn't.
- **Tail latency matters.** P50 being fine while P99 is terrible often points to garbage collection pauses, lock contention, noisy neighbors, or cold cache scenarios. Don't just measure averages.

### Concurrency Debugging

Concurrency bugs are the hardest category — they are non-deterministic, rarely reproducible, and the act of observing them (adding logging) can change their behavior (Heisenbug).

**Key heuristics:**
- **Race conditions hide behind "usually works."** If something works 99% of the time but fails 1%, suspect a race condition. The question is: what shared state is being accessed without proper synchronization?
- **Deadlocks have a signature.** Two or more threads/processes waiting for each other, each holding a resource the other needs. Check for lock ordering violations. If code acquires lock A then lock B in one path, and lock B then lock A in another, deadlock is inevitable under load.
- **Ordering assumptions are implicit contracts.** "This always runs before that" is an assumption, not a guarantee — unless enforced by synchronization primitives. When debugging async code, map out every ordering assumption and check which ones are actually enforced.
- **Reproduce with stress testing.** Concurrency bugs surface under load. Increase thread count, add artificial delays at suspected race points, run the test thousands of times. Tools like ThreadSanitizer (TSan), Helgrind, or Go's race detector can find data races mechanically.
- **Immutability eliminates classes of bugs.** If data doesn't change after creation, it can't have a race condition. When you find a concurrency bug, consider whether making the data immutable is a better fix than adding locks.

### Data and State Debugging

When the system produces wrong results but doesn't crash, the problem is usually in the data — wrong state, stale state, or inconsistent state.

**Key heuristics:**
- **Find the first wrong value.** Trace backward from the wrong output to find where the data first became incorrect. Was it wrong in the database? Wrong after transformation? Wrong at input? The first wrong value points to the owning layer.
- **Check for stale data.** Caches, replicas, materialized views, and denormalized copies all go stale. If the "source of truth" is correct but the system behaves wrong, an intermediate copy is stale.
- **Schema drift.** When code and database schema evolve independently (especially during migrations), fields can be misinterpreted, default values can change meaning, and null handling can break.
- **Time-dependent data.** Data that was correct when written may be incorrect now. Timezone handling, expiration logic, and relative-time calculations are common sources of subtle data bugs.

### Build, Dependency, and Tooling Debugging

Build failures and dependency issues are a distinct category — the system doesn't even run, and the error messages often point to tooling internals rather than your code. These require a different mental model than runtime debugging.

**Key heuristics:**
- **Read the full error, not just the last line.** Build tools often produce cascading errors. The real cause is usually the first error, not the last. Scroll up.
- **Distinguish between your code errors and toolchain errors.** A type error in your code is fundamentally different from a compilation error in a dependency or a misconfigured build tool. The fix path is completely different for each.
- **Version conflicts are combinatorial.** When dependency A requires library X v2 and dependency B requires library X v3, the resolution depends on the package manager's strategy. Understand whether your package manager deduplicates, hoists, or isolates. Check lock files — the resolved version may not be what the version range suggests.
- **"Works on my machine"** almost always means environment difference. Systematically diff: OS version, runtime version, package manager version, installed global tools, environment variables, and lock file state. The difference is the bug.
- **Stale artifacts cause phantom bugs.** Cached build outputs, stale compiled files, outdated lock files, and docker layer caches all cause situations where the source code is correct but the running artifact is wrong. When debugging makes no sense, clean all caches and rebuild from scratch before investing more time.
- **Monorepo-specific**: if a build fails after a seemingly unrelated change in another package, check for shared dependencies, build order, and workspace hoisting. The coupling is real even if the packages look independent.

### Configuration and Environment Debugging

When the code is correct but the system behaves wrong, the problem often lives outside the code — in configuration, environment variables, infrastructure, or external service state.

**Key heuristics:**
- **Print the actual config.** Don't assume the config file you're reading is the one the application loaded. Log the resolved configuration at startup. Environment variables may override file config. Multiple config sources may merge in unexpected order.
- **Environment variable precedence.** `.env` files, shell exports, Docker Compose env, Kubernetes ConfigMaps, CI/CD variables — these stack and override in platform-specific order. If a variable has the wrong value, trace the entire precedence chain.
- **Feature flags and A/B tests.** If behavior differs between environments or users, check whether a feature flag or experiment is active. These are invisible to code reading — the same code path executes different behavior based on runtime configuration.
- **External service state.** If your system integrates with external services, the external service may have changed (API version, rate limits, authentication requirements, data format) without your code changing. Check external service changelogs and status pages before blaming your code.
- **DNS, SSL, and network.** "Connection refused" and "timeout" often point to infrastructure (DNS resolution, certificate expiry, firewall rules, proxy configuration) rather than code. Check these before debugging application-level networking code.


## Knowing When to Escalate or Ask for Help

Debugging has diminishing returns. A key judgment is recognizing when you've exhausted what you can productively do alone and need to escalate.

**Escalate when:**
- **3+ fix attempts have failed.** If three different fixes based on three different hypotheses have all not resolved the problem, your mental model of the system is likely wrong. You need someone who understands the system better, not more time with the same wrong model.
- **Each fix exposes a new failure.** This is a structural/coupling problem, not a bug. It requires architectural thinking, not more debugging.
- **The bug is in someone else's domain.** If the root cause is in a service, library, or subsystem you don't own and don't fully understand, the person who owns it will find the fix faster than you will through exploration.
- **You can't reproduce it.** After reasonable effort, if you cannot trigger the bug on demand, you need either better observability (logs, traces, metrics from the environment where it does reproduce) or help from someone who has seen it happen.
- **The blast radius is high.** If the bug is in production and affecting users, the priority shifts from "find the perfect root cause" to "mitigate impact." Rollback, feature flag, or degrade gracefully first. Root cause analysis continues after mitigation.

**How to escalate effectively:**
- State what you know: the symptom, the evidence gathered, the hypotheses tested, and the results.
- State what you don't know: which layers you haven't investigated, what you couldn't verify.
- State what you think the next step is, even if you can't do it yourself.
- Do not hand off the problem without context. "It's broken" is not an escalation — it's an abdication.

---

## Failure Signals

Stop and reassess if you catch yourself doing any of these:

| Signal | What it means | What to do |
|---|---|---|
| Editing before restating the exact failure | Diagnosis is still implicit | State the failure in specific terms first |
| Picked the stack-trace file as owner without proof | Symptom site confused with owning layer | Trace the failure path to the behavior source |
| Skipped competing explanations | Jumped to conclusion | Check expectation, fixture, environment before blaming implementation |
| Multiple changes at once | Can't isolate what fixed it | Revert. One variable at a time. |
| Weakening assertions or increasing timeouts | Hiding the symptom, not finding the cause | Keep the assertion; find the cause |
| "It's probably X, let me try that" | Hypothesis without evidence | State the evidence first |
| Fix attempt 3+ without questioning architecture | Fix loop masking a design problem | Escalate (see "Knowing When to Escalate") |
| "I don't know why it works now, but it does" | Accidental fix, not understood | Find out why. If you didn't fix it deliberately, it will regress |
| Adapting a reference pattern without reading it fully | Copying without understanding | Read the working example completely first |
| Debugging code when the error points to tooling | Wrong layer — it's a build/config problem | Switch to build/dependency debugging approach |
| Assuming the config is what you think it is | Config/env assumption | Print/log the actual resolved configuration |
| Debugging alone for 1+ hour without progress | Diminishing returns | Escalate with what you know, what you tried, and what you think the next step is |

## Supporting Techniques

- `root-cause-tracing.md` — tracing backward through a call stack to the original trigger
- `defense-in-depth.md` — after finding root cause, adding validation at multiple layers so the bug class cannot recur
- `condition-based-waiting.md` — when the bug involves timing or async behavior
- `find-polluter.sh` — when a test fails only in a full suite run

## Examples

See `examples/symptom-vs-root-cause.md` — the most common place to stop too early.

See `examples/symptom-vs-owning-layer.md` — how a stack-trace line can be the symptom site without being the owning layer.

See `examples/fixture-bypass-detection.md` — how bypass information can appear acknowledged while being set aside.

