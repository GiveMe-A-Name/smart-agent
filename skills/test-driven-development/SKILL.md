---
name: test-driven-development
description: "Write tests before implementation code. TRIGGER when: about to implement a feature, bugfix, refactor, or behavior change. DO NOT TRIGGER when: change has no testable behavior — pure config (JSON/YAML), type/interface declarations, file moves, renames, formatting."
---

# Test-Driven Development

## Trigger Logic

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

- A test must exist before production code is written — no exceptions
- Every test must be watched to fail *for the right reason* before implementing
- Failing for the wrong reason means the test is wrong — fix the test before proceeding
- Implementation is the minimum code to pass the test — nothing more
- Refactoring happens only after green — never add behavior during refactor
- Bugs must never be fixed without a test that first reproduces the bug
- If skipping TDD, name a specific no-behavior criterion explicitly (not "it's simple")

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

## Failure Signals

Stop and revise when:
- code was written before a test exists
- a test passed immediately without any implementation (either it documents existing behavior — confirm explicitly — or the test is wrong and not testing what you think)
- you cannot explain why the test failed
- a test asserts on mock internals instead of real behavior
- you skipped TDD and named "it's simple" or "I'll add tests after" as the reason
- test setup is longer than the test body — redesign before adding more tests
- you are mocking a chain of three or more calls
- you are using `sleep()` as synchronization in async tests
- a test is flaky and you are re-running it instead of diagnosing the cause
- you are modifying legacy code without writing characterization tests first
- test fixtures are copied and pasted with minor variations instead of using factories/builders

## Before Exiting

Tests are green — but pause before closing.

Did I ignore any self-correction signals above because the tests are green now?
Am I exiting because tests are genuinely in place, or because the test suite *looks* complete enough?
