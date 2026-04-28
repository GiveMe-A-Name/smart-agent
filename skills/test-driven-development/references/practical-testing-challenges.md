# Practical Testing Challenges

**Load this reference when**: core TDD is in place and the situation involves legacy code, complex test data, async/concurrent code, or flaky tests.

---

## Testing Legacy Code

Legacy code — code without tests, or code whose behavior you cannot confidently predict from reading it — requires judgment about what testing strategy to use before you start changing things.

**Key signals:**
- **The first question is "do I understand the current behavior?"** If you cannot confidently state what the code does today (including its edge cases and error behavior), you are not ready to change it. Characterization tests — tests that capture what the code actually does, even if that behavior has bugs — answer this question. Write characterization tests when you cannot predict the impact of your change from reading the code alone. Skip them only when you can enumerate the specific inputs and expected outputs for every path your change touches — and you are willing to be held to that prediction. "The behavior is obvious" is not a skip criterion; being able to state the observable behavior precisely is.
- **Test scope should match change scope.** The temptation with legacy code is to either test nothing ("it's legacy, we'll refactor later") or test everything ("we need full coverage first"). Both are wrong. Test the behavior you are about to change and the behavior adjacent to it that could break. Code you are not touching does not need new tests — but code that shares state or callers with your change may need characterization tests to ensure you haven't broken it.
- **The code resists testing — what does that tell you?** When legacy code is hard to test, the friction is diagnostic. Hardcoded dependencies mean the code has hidden coupling. Global state means the code has implicit contracts. If you reach for a heavyweight mocking framework to make a test work, pause — the test is telling you the code needs a seam (an injection point, a configurable dependency), not more test infrastructure. Introduce the minimum refactoring to create the seam, then test through it.
- **Snapshots and approval tests are judgment calls, not defaults.** When a function produces complex output (HTML, serialized structures, reports), snapshot testing captures the whole output as a baseline. This is high leverage when the output is too complex to assert on field-by-field — but low leverage when the output is simple or when the snapshot becomes so large that reviewers rubber-stamp changes to it. Use snapshots when the cost of writing fine-grained assertions exceeds the cost of maintaining the snapshot.

---

## Test Data Management

How test data is created and managed has an outsized impact on test readability, maintainability, and reliability.

**Key signals:**
- **Factories/builders over raw fixtures.** Hardcoded JSON fixtures are brittle — every schema change breaks every fixture. Use factory functions or builder patterns that construct valid test data with sensible defaults, letting each test override only the fields it cares about.
- **Each test owns its data.** Tests that share mutable state (shared database records, shared test fixtures modified in place) create ordering dependencies and mysterious failures. Each test should create the data it needs and not depend on other tests having run first.
- **Minimal data for the assertion.** If you're testing that a function calculates tax correctly, you need an order with a price and a tax rate. You don't need a complete user profile, a shipping address, and payment history. Over-specified test data obscures what the test actually verifies.
- **Database tests use transactions or cleanup.** Tests that write to a real database must either run in a rolled-back transaction or clean up after themselves. A test that leaves data behind is a time bomb for the next test that runs.

---

## Testing Asynchronous and Concurrent Code

Async, event-driven, and concurrent code has failure modes that synchronous tests cannot catch by design.

**Key signals:**
- **Never use arbitrary sleeps/delays as synchronization.** `sleep(2)` in a test is a race condition waiting to happen — it will pass on fast machines and fail on slow CI runners. Use explicit synchronization: wait for a specific event, poll with a timeout, or use countdown latches / completion futures.
- **Test the observable outcome, not the timing.** "Event B fires after event A" should be tested as "when A completes, B has occurred" — not "B occurs 500ms after A." Timing-dependent assertions are the #1 source of flaky tests.
- **Concurrent code needs contention tests.** If code uses shared state with locks, semaphores, or atomic operations, test it under contention: multiple threads/goroutines/tasks accessing the same resource simultaneously. A single-threaded test of concurrent code proves nothing about thread safety.
- **Use deterministic schedulers when available.** Many async frameworks offer test utilities that give you explicit control over when tasks execute (e.g., fake timers, manual event loop advancement). Use these instead of real time — they make tests fast and deterministic.

---

## Managing Flaky Tests

A flaky test — one that sometimes passes and sometimes fails without any code change — is worse than no test. It trains the team to ignore test failures, which undermines the entire TDD feedback loop.

**Key signals:**
- **Quarantine is a triage decision, not a fix.** When a test is flaky, the immediate question is: does it stay in the main suite (blocking CI) or get quarantined (removed from the critical path)? Quarantine when the flakiness blocks others' work. But quarantine without a plan to fix is deletion with extra steps — set a deadline.
- **Rerun-on-failure masks the problem.** Retrying failed tests in CI makes builds pass, but it converts a loud signal (test failure) into a silent one (slower builds, occasional unexplained failures). Use rerun as a short-term unblock, not as a permanent strategy.
- **Flaky test rate signals test suite health.** If more than 1-2% of test runs encounter flaky failures, the test suite is losing its value as a reliable signal. At that point, fixing flakiness is higher priority than adding new tests — a test suite nobody trusts is a test suite nobody uses.
