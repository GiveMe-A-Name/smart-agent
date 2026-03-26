---
name: test-driven-development
description: Use before writing any implementation code — when a feature, bugfix, refactor, or behavior change is about to be coded. Invoke unless the work is confirmed to be pure configuration, type declarations, or explicitly throwaway exploration with no production intent.
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

If uncertain whether the work is low-value: default to TDD.

## Boundary

This skill owns:
- judging whether TDD applies to the current coding task
- executing the red-green-refactor cycle correctly
- catching test anti-patterns before they are introduced

This skill does not own:
- deciding what to implement — that comes from the task
- implementation strategy and architecture
- test framework setup or configuration

## The Core Judgment

**Can this code have a bug that a test would catch?**

If yes → TDD. If no → explain which no-behavior criterion applies, then skip.

This question cuts through rationalization. "It's simple" is not an answer to it. Simple code can have bugs. The question is whether there is behavior to verify.

### No-Behavior Signals (TDD optional)

Skip TDD only when the code genuinely has no testable behavior:

- **Pure configuration** — JSON, YAML, environment variables, no logic or branching
- **Type/interface declarations** — shapes data only, no runtime execution
- **Trivial forwarding** — a one-liner that calls one well-tested function with no logic, branching, or transformation of its own
- **Explicitly throwaway exploration** — code in `tmp/`, `poc/`, `scratch/` that will not reach production
- **Generated/scaffolded output** — auto-generated code, not hand-written logic

**When skipping**: name the criterion explicitly. "This is pure config because it's a JSON file with no logic" is acceptable. "This is simple" is not.

### Behavioral Signals (TDD required)

These signals mean testable behavior exists — do not skip:

- Conditionals, branches, or error handling paths
- Validation, calculations, or state transitions
- Code that other code depends on
- A bug being fixed (regression test required)
- An interface whose usability is unclear before use

### Rationalization Guard

If the thought is "too simple to test", "I'll test after", or "I already manually tested it" — stop. Those are the most common rationalizations. See `references/tdd-cycle.md` under Common Rationalizations.

## Invariants

**Iron Law: No production code without a failing test first.** Code written before a test must be deleted — not kept as reference, not adapted. Delete means delete.

- Every test must be watched fail before implementing. A test that passes immediately proves nothing.
- The failure message must match the expected failure. Failing for the wrong reason means the test is wrong.
- Write minimal code to pass the test. Do not add features or refactor beyond the test.
- Refactor only after green. Never add behavior during refactor.
- Never fix bugs without a test that first reproduces the bug.

## Execution

Follow the red-green-refactor cycle in `references/tdd-cycle.md`.

When writing mocks, test utilities, or any test infrastructure, read `references/testing-anti-patterns.md` first. The most common failures — testing mock behavior, adding test-only methods to production classes, incomplete mocks — happen before the production code is written.

## Self-Correction Signals

Stop and revise when:
- code was written before a test exists
- a test passed immediately without any implementation
- you cannot explain why the test failed
- a test is asserting on mock elements or mock call counts instead of real behavior
- you skipped TDD and named "it's simple" or "I'll test after" as the reason
- you skipped TDD without naming a no-behavior criterion
