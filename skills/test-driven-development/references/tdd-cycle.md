# TDD Cycle Reference

**Load this reference for**: red-green-refactor mechanics, core techniques, getting unstuck, and rationalizations.

---

## Red-Green-Refactor

### RED — Write a Failing Test

Write one minimal test showing what the next behavior should be, written from the caller's perspective.

**Requirements:**
- One behavior per test
- Name describes the observable behavior, not the implementation
- Written as if the code already exists and you're asserting what it should do
- Tests real behavior — mocks only when unavoidable (see `testing-anti-patterns.md`)

See `examples/test-quality.md` for a Good/Bad contrast pair.

### Verify RED — Watch It Fail

Run the test. Confirm:
- Test fails, not errors (an error means the test scaffolding is broken — fix it)
- Failure message matches the expected failure
- Fails because the feature is missing, not due to a typo or broken import

**Test passes immediately?** Either you are testing already-existing behavior (confirm this explicitly, then keep the test as documentation), or the test is wrong and not actually testing what you think. Do not proceed until you understand which.

**Test errors on import/syntax?** Fix the compilation error — it is not a valid RED state.

### GREEN — Minimal Code

Write the simplest code that makes the test pass. Do not add features, refactor other code, or "improve" anything beyond what the test requires.

**Permitted on GREEN:**
- Hardcode the return value if that passes the test (see *Fake It Till You Make It* below)
- Write the ugliest, most direct code that works

**Not permitted on GREEN:**
- Adding logic no current test requires
- Refactoring while the test is red

See `examples/test-quality.md` for a Good/Bad contrast pair.

### Verify GREEN — Watch It Pass

Run the test. Confirm:
- The new test passes
- Tests already passing before this cycle still pass
- No new errors or warnings introduced that weren't already present

**New test fails?** Fix the code, not the test.
**Previously passing test now fails?** Fix this before moving on — you broke something.

### REFACTOR — Clean Up

After green only. Permitted: remove duplication, improve names, extract helpers, reorganize structure. **Not permitted: add behavior.** Keep all tests green throughout every refactor step — if a test breaks during refactor, you accidentally changed behavior. Undo and try again.

### Repeat

Move to the next behavior. Write the next failing test.

---

## Core Techniques

### Fake It Till You Make It

On the first cycle, hardcoding the return value is correct:

```typescript
// First test: expects calculateDiscount(50) === 0
function calculateDiscount(total: number): number {
  return 0; // Hardcoded. Correct for now.
}
```

This looks wrong, but it is right. The next test will prove it insufficient and force a real implementation. Each hardcoded step is a provably correct state. Never skip ahead to "the real implementation" before a test requires it.

### Triangulation

Add multiple specific tests to force a general algorithm:

| Test added | Minimum implementation forced |
|------------|-------------------------------|
| `discount(50) === 0` | `return 0` |
| `discount(150) === 15` | `if total > 100: return total * 0.1` |
| `discount(250) === 50` | add `if total > 200: return total * 0.2` |

Triangulation is the mechanical process of forcing generalization without guessing. When you don't know what the full algorithm looks like yet, triangulation lets you build toward it one verified step at a time.

### One Cycle at a Time

Resist writing multiple failing tests before implementing. The discipline of one cycle (RED → GREEN → REFACTOR) at a time keeps each step small enough that failures are instantly diagnosable. The moment you have two failing tests, you lose the clean signal that tells you exactly what broke.

**Signs you are testing too much in a single cycle:**
- The test name contains "and"
- Assertions cover multiple independent concerns
- The test exercises multiple execution paths simultaneously

Split it. A feature is a series of short TDD cycles. Each cycle should be completable in minutes, not hours.

**What constitutes a behavior**: any observable change a caller could depend on — a return value, a side effect, an error thrown, a state transition. Each behavior is one cycle.

---

## Design Pressure

Hard tests are design feedback, not a TDD limitation.

If you find yourself in any of these states while trying to write a test, stop writing the test:

| State | What TDD is telling you |
|-------|------------------------|
| Setup code is longer than the assertion | The class under test has too many responsibilities |
| Cannot control a dependency without `new` in the constructor | Inject the dependency instead of constructing it |
| Must chain three or more mocks to get to the thing you want to assert | Law of Demeter violation — logic is leaking across objects |
| The test is tightly coupled to the call sequence inside the method | You are testing implementation, not behavior — extract the interface |
| You need the real database/network to run a simple unit test | Missing an abstraction seam at the infrastructure boundary |

**The response to friction is redesign, not a more complex test.** The test is showing you the design you should have had.

---

## Completion Signals

Work is complete when:
- Every behavior added has a test that was watched fail before the code existed
- All tests pass
- Test output introduced by this change is clean
- Tests verify real observable behavior, not mock behavior or implementation details

If you cannot explain why each test was red before the implementation, TDD was not followed.

---

## When Stuck on RED

| Problem | Resolution |
|---------|------------|
| Don't know how to assert the behavior | Write the assertion you *wish* you could write. If the API doesn't exist yet, design it to make that assertion possible. |
| Test setup is impossibly complex | The design is too coupled. Simplify it before writing more tests. |
| Must mock everything to isolate the unit | The code is too coupled. Inject dependencies. |
| Can't figure out what the next test should be | Break the feature into smaller behaviors. What is the *simplest* thing that could possibly go wrong? |
| Writing the test reveals the requirement is ambiguous | Good — surface the ambiguity now, not after implementing. Ask, don't guess. |

---

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "Too simple to test" | Simple code breaks. Writing the test takes 30 seconds. |
| "I'll test after" | Tests written after existing code pass immediately — they test what was built, not what should have been built. |
| "Tests-after achieve the same goals" | Tests-after answer "what does this do?" Tests-first answer "what should this do?" These produce different tests. |
| "Already manually tested it" | Manual testing is not repeatable, not recorded, not re-runnable under pressure. |
| "Deleting work is wasteful" | Sunk cost. Keeping unverified code is technical debt. |
| "Keep it as reference, write tests first" | You will adapt it. That is testing after. Delete means delete. |
| "Need to explore first" | Exploration is fine. Throw away the exploration. Start TDD clean. |
| "TDD will slow me down" | TDD is faster than debugging. Time lost writing tests is recovered in not hunting bugs. |
| "This is different because..." | Stop. That sentence ends in rationalization. |
