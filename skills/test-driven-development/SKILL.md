---
name: test-driven-development
description: "Invoke before writing any implementation code for a feature, bugfix, refactor, or behavior change. Cost of unnecessary invocation: a short judgment pass. Cost of missing: untested code, silent regressions, and design problems discovered too late to fix cheaply. When uncertain whether the work has testable behavior, invoke."
---

# Test-Driven Development

## Trigger Logic

**Invocation default**: Missing TDD guidance produces untested code and silent regressions. The cost of invoking unnecessarily is a short judgment pass. Invoke before writing any implementation code unless the work is confirmed to fall into a no-behavior category.

Use this skill when:
- implementation work is about to begin on a feature, bugfix, or refactor
- a behavior change is being made to existing code
- a bug is being fixed

Do not use this skill when:
- the work is confirmed to be pure configuration with no executable logic
- the work is type or interface declarations only, with no runtime behavior
- the change is structural only: file move, import reorder, rename with no behavioral effect

If uncertain whether the work falls into a no-behavior category: default to TDD.

## Boundary

This skill owns:
- judging whether TDD applies to the current task
- deciding the right orientation and granularity before writing the first test
- executing the red-green-refactor cycle with design intention
- reading design pressure signals that tests surface before implementation exists

This skill does not own:
- deciding what to implement — that comes from the task
- test framework selection or configuration
- overall system architecture

## The Central Mental Model

**TDD is not primarily a testing technique. It is a design technique that produces tests as a byproduct.**

Writing the test first forces one irreplaceable act: you must use the API before you build it. You become the first consumer of your own design. The test exposes API awkwardness, unclear responsibilities, and hidden dependencies before a line of production code exists.

This is the primary value — not coverage, not regression safety (though those follow), but *design feedback applied before anything is committed*.

A well-applied TDD session answers: what should this thing do, from the perspective of the code that will call it? That question, made concrete as an assertion, is the specification.

**The core judgment**: Can this code have a bug that a test would catch? If yes → TDD. If no → name a specific no-behavior criterion, then skip. "It's simple" is not a no-behavior criterion. Simple code can have bugs.

## Invariants

- A test must exist before production code is written — no exceptions
- Every test must be watched to fail *for the right reason* before implementing; a test that passes immediately proves nothing
- Failing for the wrong reason means the test is wrong — fix the test before proceeding
- Implementation is the minimum code to pass the test — nothing more
- Refactoring happens only after green — never add behavior during refactor
- Bugs must never be fixed without a test that first reproduces the bug
- If skipping TDD, name a specific no-behavior criterion explicitly (not "it's simple")

## Completion Criteria

- [ ] All invariants were followed (test before code, watched it fail, watched it pass).
- [ ] If TDD was skipped, a specific no-behavior criterion was named.
- [ ] Each test names a behavior the caller cares about, not an implementation detail.
- [ ] If TDD surfaced design pressure, that pressure was acted on rather than rationalized away.
- [ ] If modifying legacy code, characterization tests were written before changing behavior.
- [ ] If testing async or concurrent code, explicit synchronization is used instead of sleeps.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the test discipline was genuinely applied.

Did I ignore any applicable self-correction signals because the tests are green now?

Am I exiting because tests are genuinely in place, or because the current test suite looks reassuring enough?

### No-Behavior Signals (TDD optional)

Skip TDD only when the code genuinely has no testable behavior:

- **Pure configuration** — JSON, YAML, env vars, no logic or branching
- **Type/interface declarations** — shapes data only, no runtime execution
- **Trivial forwarding** — a one-liner delegating to one well-tested function with no branching or transformation
- **Explicitly throwaway exploration** — code in `tmp/`, `poc/`, `scratch/` that will not reach production
- **Generated/scaffolded output** — auto-generated code, not hand-written logic

When skipping: name the criterion explicitly. "Pure config — JSON file with no logic" is acceptable. "Simple" is not. See `examples/low-value-judgment.md` for contrast cases.

## Decision Signals

These signals guide judgment before and during TDD execution. They are not steps.

### Choose Your Orientation First

Before writing the first test, decide: should you work outside-in or inside-out?

**Outside-in (London school)**: Start at the outermost interface the caller will use. Test the public behavior, use test doubles for collaborators that don't yet exist. Work inward layer by layer.

*Use when:* interface or API design is the hard part; the feature spans multiple layers; you need to discover what the right seam is before building the internals.

**Inside-out (Chicago school)**: Start from the core domain object or algorithm. Get the inner logic working in isolation first, then integrate outward.

*Use when:* the algorithm or data transformation is the hard part; collaborators already exist and are well-tested; you're building a well-defined calculation or rule engine.

Neither is a rule. Most real features use both: outside-in to find the right interface, inside-out for complex inner logic. The London school reduces over-engineering risk; the Chicago school reduces mock-coupling risk.

### Read Design Pressure Signals

If writing the test is *hard*, the design is wrong. This is the most important diagnostic TDD provides.

| Friction when writing the test | Design signal |
|-------------------------------|---------------|
| Setup longer than the test body | Class has too many responsibilities |
| Must mock deeply chained calls (`a.b().c().d()`) | Law of Demeter broken; logic leaking across boundaries |
| Cannot inject a dependency to control behavior | Class creates its own collaborators (`new` inside logic) |
| Test needs to know implementation internals | Behavior not encapsulated; testing structure, not contract |
| Every test requires real infrastructure | Missing an abstraction at the system boundary |

These signals appear as friction *before* the code exists. That friction is TDD telling you to redesign. Do not work around friction with more complex tests — fix the design.

### Slice Work Into Single Behaviors

One TDD cycle tests exactly one behavior — one reason a test can fail. Signs you're trying to test too much at once:

- The test name contains "and"
- Assertions cover multiple independent concerns
- The test exercises multiple execution paths simultaneously

Split it. A feature is a series of short TDD cycles. Each cycle should be completable in minutes, not hours.

**What constitutes a behavior**: any observable change a caller could depend on — a return value, a side effect, an error thrown, a state transition. Each behavior is one cycle.

### Know When to Use "Fake It Till You Make It"

In the first cycle, the simplest implementation might be to hardcode the answer. That's correct. The next test will force you to generalize. This technique (called triangulation) keeps each step minimal and each failure message meaningful.

Return a constant → second test forces a condition → third test forces a loop. Each step is provably correct at that moment.

### Choose the Right Test Level

Write the test at the level that gives the most useful failure message without requiring you to build too much first:

- Pure function / algorithm → unit test directly
- Behavior across a few cooperating classes → test through the public interface
- System boundary (HTTP, DB, file I/O) → test through a thin seam; use acceptance tests for critical paths

The right level is the one where a test failure tells you exactly what broke — no more setup, no less coverage.

## Execution

Red-green-refactor. Repeat per behavior.

1. **RED**: Write one test for the next behavior. Run it. Watch it fail for the right reason.
2. **GREEN**: Write the minimum code to pass. Run it. Watch it pass.
3. **REFACTOR**: Remove duplication, improve names. No new behavior. All tests stay green throughout.
4. Repeat for the next behavior.

See `references/tdd-cycle.md` for: cycle mechanics, triangulation technique, stuck-on-red patterns, and common rationalizations.

See `references/testing-anti-patterns.md` for mock discipline — read before writing any test infrastructure.

See `examples/complete-example.md` for a full worked walkthrough: from requirement to passing feature, including orientation decision, design pressure, triangulation, and refactor.

## Advanced Testing Strategies

The core TDD cycle (red-green-refactor) is the foundation. These strategies extend it for specific situations where example-based unit tests alone are insufficient.

### Property-Based Testing

Instead of specifying individual input-output pairs, specify properties that should hold for ALL valid inputs. The test framework generates hundreds or thousands of random inputs automatically.

**When to reach for it:**
- The input space is large or combinatorial (parsers, serializers, data transformations)
- You find yourself writing many similar test cases that differ only in input values
- The code must satisfy mathematical properties (idempotency, commutativity, roundtrip encoding/decoding)
- Edge cases are hard to enumerate manually

**Key signals:**
- A property like "serialize then deserialize returns the original value" catches bugs that no finite set of examples would find
- "For any valid input, the output satisfies [invariant]" is more powerful than testing 5 specific inputs
- When a property-based test finds a failure, it shrinks the input to the minimal failing case — this IS the reproduction
- Property-based tests complement, not replace, example-based tests. Use examples for documentation and specific edge cases; use properties for broad correctness guarantees

### Contract Testing

When your code communicates across a boundary (API, message queue, shared library), both sides have expectations about the data format and behavior. Contract tests verify that these expectations remain aligned.

**When to reach for it:**
- Your code consumes an API owned by another team or service
- Your code provides an API consumed by others
- Integration tests are slow, flaky, or require complex environment setup
- You've been bitten by "it works in my test but breaks when the real service responds differently"

**Key signals:**
- Consumer-driven contracts: the consumer defines what it expects; the provider verifies it can meet those expectations. This catches breaking changes before deployment.
- Contract tests are NOT integration tests. They run fast, use no network, and verify the contract (data shape, required fields, error codes) independently of implementation.
- When adding a new field to an API response, contract tests tell you which consumers will break — before you deploy.

### Mutation Testing

Mutation testing answers the question: "If a bug existed in my code, would my tests actually catch it?" It works by making small changes (mutations) to the production code and checking if any test fails.

**When to reach for it:**
- You want to assess the quality of your test suite, not just coverage
- Code coverage is high but you suspect the tests are weakly asserted
- You are refactoring and want confidence that the test suite is a genuine safety net

**Key signals:**
- A surviving mutant (a code change that no test catches) reveals a gap in your test suite — either a missing test or a weak assertion
- 100% line coverage with 60% mutation score means 40% of potential bugs would go undetected
- Mutation testing is expensive to run. Use it periodically on critical code paths, not on every commit
- Focus on mutants that survive in business logic, not in boilerplate or glue code

### Testing Beyond Unit Tests

TDD starts at the unit level, but complete test confidence requires thinking about the full testing pyramid:

**Integration tests** verify that real components work together. Use them for: database interactions, external API calls (through a real test instance, not mocks), message queue behavior, file system operations.

**End-to-end tests** verify user-facing scenarios through the actual system. Use sparingly — they are slow and brittle. Each e2e test should map to a critical user journey that, if broken, would be immediately noticed.

**Testing in production** is not testing in the traditional sense — it's observability plus controlled rollout:
- Feature flags allow testing new behavior with a subset of users
- Canary deployments compare new code against old code with real traffic
- A/B tests verify that a change produces the expected outcome at scale
- These do not replace pre-deployment testing — they catch the things that pre-deployment testing cannot (scale effects, real user behavior, production data patterns)

## Practical Testing Challenges

The core TDD cycle and advanced strategies above cover the ideal case. Real codebases present situations where applying TDD requires additional judgment.

### Testing Legacy Code

Legacy code — code without tests, or code whose behavior you cannot confidently predict from reading it — requires judgment about what testing strategy to use before you start changing things.

**Key signals**:
- **The first question is "do I understand the current behavior?"** If you cannot confidently state what the code does today (including its edge cases and error behavior), you are not ready to change it. Characterization tests — tests that capture what the code actually does, even if that behavior has bugs — answer this question. The judgment: write characterization tests when you cannot predict the impact of your change from reading the code alone. Skip them when the behavior is obvious and the change is well-bounded.
- **Test scope should match change scope.** The temptation with legacy code is to either test nothing ("it's legacy, we'll refactor later") or test everything ("we need full coverage first"). Both are wrong. Test the behavior you are about to change and the behavior adjacent to it that could break. Code you are not touching does not need new tests — but code that shares state or callers with your change may need characterization tests to ensure you haven't broken it.
- **The code resists testing — what does that tell you?** When legacy code is hard to test, the friction is diagnostic. Hardcoded dependencies mean the code has hidden coupling. Global state means the code has implicit contracts. If you reach for a heavyweight mocking framework to make a test work, pause — the test is telling you the code needs a seam (an injection point, a configurable dependency), not more test infrastructure. Introduce the minimum refactoring to create the seam, then test through it.
- **Snapshots and approval tests are judgment calls, not defaults.** When a function produces complex output (HTML, serialized structures, reports), snapshot testing captures the whole output as a baseline. This is high leverage when the output is too complex to assert on field-by-field — but low leverage when the output is simple or when the snapshot becomes so large that reviewers rubber-stamp changes to it. Use snapshots when the cost of writing fine-grained assertions exceeds the cost of maintaining the snapshot.

### Test Data Management

How test data is created and managed has an outsized impact on test readability, maintainability, and reliability.

**Key signals**:
- **Factories/builders over raw fixtures.** Hardcoded JSON fixtures are brittle — every schema change breaks every fixture. Use factory functions or builder patterns that construct valid test data with sensible defaults, letting each test override only the fields it cares about.
- **Each test owns its data.** Tests that share mutable state (shared database records, shared test fixtures modified in place) create ordering dependencies and mysterious failures. Each test should create the data it needs and not depend on other tests having run first.
- **Minimal data for the assertion.** If you're testing that a function calculates tax correctly, you need an order with a price and a tax rate. You don't need a complete user profile, a shipping address, and payment history. Over-specified test data obscures what the test actually verifies.
- **Database tests use transactions or cleanup.** Tests that write to a real database must either run in a rolled-back transaction or clean up after themselves. A test that leaves data behind is a time bomb for the next test that runs.

### Testing Asynchronous and Concurrent Code

Async, event-driven, and concurrent code has failure modes that synchronous tests cannot catch by design.

**Key signals**:
- **Never use arbitrary sleeps/delays as synchronization.** `sleep(2)` in a test is a race condition waiting to happen — it will pass on fast machines and fail on slow CI runners. Use explicit synchronization: wait for a specific event, poll with a timeout, or use countdown latches / completion futures.
- **Test the observable outcome, not the timing.** "Event B fires after event A" should be tested as "when A completes, B has occurred" — not "B occurs 500ms after A." Timing-dependent assertions are the #1 source of flaky tests.
- **Concurrent code needs contention tests.** If code uses shared state with locks, semaphores, or atomic operations, test it under contention: multiple threads/goroutines/tasks accessing the same resource simultaneously. A single-threaded test of concurrent code proves nothing about thread safety.
- **Use deterministic schedulers when available.** Many async frameworks offer test utilities that give you explicit control over when tasks execute (e.g., fake timers, manual event loop advancement). Use these instead of real time — they make tests fast and deterministic.

### Managing Flaky Tests

A flaky test — one that sometimes passes and sometimes fails without any code change — is worse than no test. It trains the team to ignore test failures, which undermines the entire TDD feedback loop.

**Key signals**:
- **Quarantine is a triage decision, not a fix.** When a test is flaky, the immediate question is: does it stay in the main suite (blocking CI) or get quarantined (removed from the critical path)? Quarantine when the flakiness blocks others' work. But quarantine without a plan to fix is deletion with extra steps — set a deadline.
- **Rerun-on-failure masks the problem.** Retrying failed tests in CI makes builds pass, but it converts a loud signal (test failure) into a silent one (slower builds, occasional unexplained failures). Use rerun as a short-term unblock, not as a permanent strategy.
- **Flaky test rate signals test suite health.** If more than 1-2% of test runs encounter flaky failures, the test suite is losing its value as a reliable signal. At that point, fixing flakiness is higher priority than adding new tests — a test suite nobody trusts is a test suite nobody uses.

## Self-Correction Signals

Stop and revise when:
- code was written before a test exists
- a test passed immediately without any implementation (verify it's documenting existing behavior, not failing to test new behavior)
- you cannot explain why the test failed
- a test asserts on mock internals (call counts, mock state) instead of real behavior
- you skipped TDD and named "it's simple" or "I'll add tests after" as the reason
- test setup is longer than the test body — redesign before adding more tests
- you are mocking a chain of three or more calls to make a test work
- you are using `sleep()` or arbitrary delays as synchronization in async tests
- a test is flaky and you are re-running it instead of diagnosing the cause
- you are modifying legacy code without writing characterization tests first
- test fixtures are copied and pasted with minor variations instead of using factories/builders
- review feedback says "the goal is not fully achieved" and you are adding more tests toward that goal without first verifying that the goal itself was explicitly confirmed by the user — a reviewer's articulation of the goal is not the same as the user's confirmed intent; writing more tests to satisfy an unverified goal deepens the implementation of a potentially wrong objective

