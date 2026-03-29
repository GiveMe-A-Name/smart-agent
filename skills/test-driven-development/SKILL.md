---
name: test-driven-development
description: Invoke before writing any implementation code unless the work is confirmed to be pure configuration, type declarations, or explicitly throwaway exploration with no production intent. When a feature, bugfix, refactor, or behavior change is about to be coded, invoke. When uncertain whether tests are needed, invoke.
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
- **Before exiting this skill, complete the Self-Check section**

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

## Self-Correction Signals

Stop and revise when:
- code was written before a test exists
- a test passed immediately without any implementation (verify it's documenting existing behavior, not failing to test new behavior)
- you cannot explain why the test failed
- a test asserts on mock internals (call counts, mock state) instead of real behavior
- you skipped TDD and named "it's simple" or "I'll add tests after" as the reason
- test setup is longer than the test body — redesign before adding more tests
- you are mocking a chain of three or more calls to make a test work

## Self-Check Before Exiting

- [ ] Did I follow all invariants (test before code, watched it fail, watched it pass)?
- [ ] If I skipped TDD, did I name a specific no-behavior criterion?
- [ ] Did each test name a behavior the caller cares about — not an implementation detail?
- [ ] Did TDD surface any design pressure? Did I act on it or rationalize past it?
- [ ] Did I catch any self-correction signals?
- [ ] Am I exiting because tests are genuinely in place, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
