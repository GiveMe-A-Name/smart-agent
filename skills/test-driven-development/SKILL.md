---
name: test-driven-development
description: "Write tests before implementation code. TRIGGER when: about to implement a feature, bugfix, refactor, or behavior change. DO NOT TRIGGER when: change has no testable behavior — pure config (JSON/YAML), type/interface declarations, file moves, renames, formatting."
---

# Test-Driven Development

## Trigger Logic

**Invocation default**: The cost of invoking TDD unnecessarily is writing one test before discovering the code has no testable behavior. The cost of skipping TDD on code that has behavior is losing design feedback permanently — the API gets built before anyone uses it, and structural problems surface in integration rather than during design.

Use when: implementation work is about to begin on a feature, bugfix, or refactor — or a behavior change is being made.

Skip when the code genuinely has no testable behavior:
- **Pure configuration** — JSON, YAML, env vars, no logic or branching
- **Type/interface declarations** — shapes data only, no runtime execution
- **Structural change only** — file move, import reorder, rename with no behavioral effect
- **Trivial forwarding** — a one-liner delegating to a single well-tested function, no branching
- **Throwaway exploration** — code in `tmp/`, `poc/`, `scratch/` that will not reach production
- **Generated/scaffolded output** — auto-generated code, not hand-written logic

If uncertain: default to TDD. See `examples/low-value-judgment.md` for contrast cases.

## Boundary

This skill owns:
- judging whether TDD applies to the current task
- deciding orientation and granularity before writing the first test
- executing the red-green-refactor cycle with design intention
- reading design pressure signals that tests surface before implementation exists

This skill does not own:
- deciding what to implement — that comes from the task
- test framework selection or configuration
- overall system architecture

## Core Principle

TDD is a design technique, not a testing technique. Writing the test first forces you to use the API before building it — you become the first consumer of your own design. Design feedback arrives before any code is committed.

**Core judgment**: Can this code have a bug a test would catch? Yes → TDD. No → name the specific no-behavior criterion. "It's simple" is not a criterion.

## Invariants

- A test must exist before production code is written — no exceptions [because implementation without a prior failing test means the design was never used before it was built; structural problems surface at integration, not at design time]
- Every test must be watched to fail *for the right reason* before implementing [because a test that fails for the wrong reason may pass after implementation without actually covering the intended behavior]
- Failing for the wrong reason means the test is wrong — fix the test before proceeding
- Implementation is the minimum code to pass the test — nothing more [because over-implementation bypasses the next test, losing design signal and leaving behavior untested]
- Refactoring happens only after green — never add behavior during refactor [because behavior added during refactor has no prior failing test, silently violating the first invariant]
- Bugs must never be fixed without a test that first reproduces the bug [because without a reproducing test, the fix cannot be verified to address the root cause and the bug may recur silently]
- If skipping TDD, name a specific no-behavior criterion explicitly (not "it's simple") [because "it's simple" cannot be verified by others and is the canonical rationalization for skipping test-first discipline]

## Orientation

Before the first test, decide:
- **Outside-in**: Start at the public interface, use test doubles for absent collaborators, work inward. *Use when interface design is the hard problem.*
- **Inside-out**: Start from core domain logic, integrate outward. *Use when the algorithm is the hard problem.*

Most features use both. See `references/tdd-orientation.md` for London/Chicago school detail and trade-offs.

## Execution

RED → GREEN → REFACTOR. One behavior per cycle.

1. **RED**: Write one test for the next behavior. Run it. Watch it fail for the right reason.
2. **GREEN**: Write the minimum code to pass. Run it. Watch it pass.
3. **REFACTOR**: Remove duplication, improve names. No new behavior. All tests stay green.
4. Repeat for the next behavior.

See `references/tdd-cycle.md` for cycle mechanics, triangulation, design pressure signals, and stuck-on-red patterns.
See `references/tdd-orientation.md` for London/Chicago school detail and trade-offs.
See `references/testing-anti-patterns.md` for mock discipline — read before writing any test infrastructure.
See `references/advanced-testing-strategies.md` for property-based, contract, and mutation testing.
See `references/practical-testing-challenges.md` for legacy code, async/concurrent code, and flaky tests.
See `examples/complete-example.md` for a full worked walkthrough.

## Completion Criteria

- [ ] Every behavior added during this task has a corresponding test that was written and seen to fail before implementation began (verifiable from conversation history)
- [ ] All tests pass
- [ ] Every TDD skip names a specific no-behavior criterion — not "it's simple", "I'll add tests after", or "this is different" (verifiable from conversation history)
- [ ] No refactor pass introduced new behavior — tests were already green before each refactor began (verifiable from conversation history)
- [ ] You can explain why each test was red before implementation — not just that it was red (verifiable from conversation history)

**If any criterion is not met, return to the relevant cycle before exiting.**

## Verification Approach

This skill is artifact-type: completion is self-checked against the Completion Criteria above. State conditions are answered from conversation history; artifact conditions are answered from test run output. No user confirmation is required.

## Failure Signals

Stop and revise when:
- code was written before a test exists
- a test passed immediately without any implementation (either it documents existing behavior — confirm explicitly — or the test is wrong and not testing what you think)
- you cannot explain why the test failed [the test may not be exercising the intended path; implementation may pass the test without covering the real behavior]
- a test asserts on mock internals instead of real behavior [mocks verify call sequences, not outcomes — the test can pass while the actual behavior is wrong]
- you skipped TDD and named "it's simple" or "I'll add tests after" as the reason
- test setup is longer than the test body [excessive setup signals too many dependencies in the unit under test, making bugs harder to locate and future tests harder to write — redesign before adding more tests]
- you are mocking a chain of three or more calls [chained mocks test your mock setup, not your code; any refactor of the chain breaks the test without catching a real bug]
- you are using `sleep()` as synchronization in async tests [sleep-based synchronization is timing-dependent, creating flaky tests that mask real concurrency bugs behind noise]
- a test is flaky and you are re-running it instead of diagnosing the cause [flaky tests erode trust in the suite and can mask real failures; re-running treats the symptom, not the cause]
- you are modifying legacy code without writing characterization tests first [without characterization tests, existing behavior cannot be verified as preserved; regressions become invisible]
- test fixtures are copied and pasted with minor variations instead of using factories/builders [duplicated fixtures diverge silently; a change to one copy does not propagate, making tests misrepresent what they cover]

## Anti-Rationalization Check

This section exists because the moment tests are green is exactly when the risk of premature exit is highest — the work looks done.

Did I ignore any failure signal above because the tests are green now?

Am I exiting because TDD was genuinely applied to every behavior, or because the test suite *looks* complete enough?
