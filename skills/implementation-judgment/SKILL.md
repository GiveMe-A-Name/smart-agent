---
name: implementation-judgment
description: Use before writing any code to decide how to land the change — not just where. Knowing the change point does not answer whether the approach is right. Invoke unless the work is purely mechanical with no behavioral or structural implications.
---

# Implementation Judgment

Implement changes with engineering judgment, not just task completion.

This skill applies engineering judgment to code changes — not just locating where to change, but reasoning about how the change should land given prior decisions, existing constraints, and long-term maintainability. The value is in the judgment: understanding why code is shaped the way it is, whether current abstractions are helping or hurting, where complexity should live, how the code will fail, and what the change means for callers and future maintainers.

## Trigger Logic

**Invocation default**: The trigger state is simple: code is about to be written. The cost of an unnecessary invocation is a few extra reasoning steps. The cost of a missed invocation is a patch that weakens structure, moves responsibility into the wrong layer, introduces a subtle failure mode, or locks in a wrong abstraction — often invisibly. Invoke unless the work is purely mechanical with no behavioral effect.

**Knowing where to change is not the same as knowing how.** Having a plan, a root cause, or a change point identified answers the first question. Whether the approach is the right one — given tradeoffs, failure modes, and what the current code is protecting — is a separate question. Prior investigation, planning, or root-cause analysis does not substitute for this judgment — they ground it.

**Prerequisites**: This skill requires code evidence and clarified intent to reason responsibly. If code evidence is still missing, build that understanding first and return here. If intent is still unclear, clarify that first and return here. These are sequencing dependencies — not reasons to skip this skill, but work to complete before this skill can be useful.


## Boundary

This skill owns:
- reasoning from clarified intent and grounded code evidence to responsible implementation
- judging whether candidate abstractions, patterns, and boundaries should be preserved, extended, corrected, or removed
- deciding where complexity should live and who should own each responsibility
- evaluating failure modes, edge cases, and implicit contracts before writing code
- deciding whether a local patch is sufficient, a focused structural improvement is warranted, or an explicit pause is needed
- balancing delivery speed, simplicity, system coherence, and future maintainability

This skill does not own:
- product clarification from scratch
- repository analysis as an end in itself — if code evidence is missing, that work belongs before this skill, not inside it
- generic task planning
- unrestricted refactoring beyond what the current change needs
- final completion or release judgment


## Invariants

- Do not treat the task description as the full implementation truth.
- Do not assume existing code shape is arbitrary before considering what it may be protecting.
- Do not infer unconfirmed product intent from code shape alone.
- Do not let deadline or delivery pressure justify moving responsibility into the wrong layer.
- Prefer the least invasive change that keeps responsibilities clear and future changes easier — but recognize that sometimes the "least invasive" change deepens a structural problem.
- Do not treat theoretical justification as a substitute for usage evidence. "This supports testability", "this could be useful", "this is clean design" are arguments, not evidence. Evidence is: who calls this, who depends on this, what breaks if this is removed.
- Do not treat duplication removal as inherently good. Duplication is far cheaper than the wrong abstraction.
- **Before exiting this skill, you MUST complete the Self-Check section at the end.**


## Judgment Dimensions

When reasoning about how a change should land, work through these dimensions. Not every dimension applies to every change — but skipping a relevant one is how subtle damage happens.

### 1. Abstraction Correctness

**Question**: Are the existing abstractions helping or hurting? Should this change create, extend, preserve, or undo an abstraction?

**Key signals**:
- If an abstraction requires parameters and conditional branches to handle different cases, it may be the wrong abstraction. The fastest way forward is often back — inline the abstraction, let the callers diverge, then see what new pattern emerges.
- If code is duplicated in 2 places and the copies are diverging, keep them separate. Premature unification creates a coupling point that makes both cases harder to change.
- If code is duplicated in 3+ places with identical semantics and the same rate of change, extraction is likely warranted.
- If you're about to "clean up" repetition, ask: are these cases truly the same, or do they just look the same right now? Code that looks similar but changes for different reasons should not share an abstraction.
- If an existing shared abstraction has accumulated conditionals to handle special cases, consider whether inlining it back would make each caller simpler and more honest about what it actually does.

See `examples/wrong-abstraction.md` for a case where eagerly extracting surface-similar code creates a worse outcome than tolerating duplication. See `examples/security-review-duplicate-helper.md` for the opposite case — where 3+ identical copies of security logic should be consolidated into a shared helper.

### 2. Complexity Placement

**Question**: Which layer should own this complexity? Is the change pushing complexity toward the right boundary or scattering it?

**Key signals**:
- Deep modules are better than shallow modules. A module with a simple interface that hides complex implementation is more valuable than one with a complex interface and trivial implementation.
- If callers need to know implementation details to use a function correctly, the abstraction boundary is in the wrong place.
- Pull complexity downward: if you can make 5 callers simpler by making 1 implementation more complex, do it. The net cognitive load decreases.
- When adding a configuration parameter or flag, ask: could this decision be made automatically inside the module instead of pushed to every caller? Each configuration parameter is complexity exported to every user.
- Validation, transformation, and error handling for a particular concern should live together in one layer, not be scattered across multiple layers with each layer doing a partial job.

See `examples/complexity-misplacement.md` for a case where partial validation spread across layers creates a false sense of safety.

### 3. Failure Mode Awareness

**Question**: How will this code fail? Are the failure paths explicit and handled at the right level?

**Key signals**:
- Error handling is most effective at the ends (end-to-end principle). If every intermediate layer handles errors partially, you end up with duplicated, inconsistent recovery logic and errors that get swallowed or transformed beyond recognition.
- When adding error handling, ask: who is the right party to make the recovery decision? Often it is the outermost caller who has the full context, not the inner function that detected the problem.
- Distinguish between errors that should be retried, errors that should be surfaced to the user, and errors that indicate programmer mistakes. These require different handling strategies — conflating them leads to both swallowed bugs and noisy false alarms.
- When a function can fail in ways the caller doesn't check, the code has an implicit failure path. Make failure paths explicit, even if the handling is "propagate to the caller."
- Consider partial failure: if this operation does steps A, B, C and fails at B, what state is the system in? Is that state recoverable?

### 4. Deletability and Replaceability

**Question**: If requirements change, how easy is it to delete or replace this code? Is the change making the codebase more or less entangled?

**Key signals**:
- Code that is easy to delete is better than code that is easy to extend. Extension assumes you know the future; deletion works regardless.
- Isolate the parts that are likely to change from the parts that are likely to stay stable. Business rules change fast; data structures change slowly; protocols change slowest.
- If deleting this code would require changes in many unrelated modules, the coupling is too tight. Loose coupling means you can change your mind without rewriting the world.
- Feature flags, runtime configuration, and clear module boundaries all increase deletability. Deep inheritance hierarchies, global state, and implicit dependencies decrease it.
- When building something new and uncertain, favor a "big lump" that can be deleted as a whole over many small interlocking pieces that can only be deleted piecewise.

### 5. Contract and Compatibility Awareness

**Question**: What does this change mean for callers? Are implicit contracts being preserved or silently broken?

**Key signals**:
- Callers depend on observed behavior, not documented behavior. If a function has always returned an empty array on error (even by accident), callers may rely on that. Changing it to throw is a breaking change regardless of what the docs say.
- Adding a parameter with a default value looks safe but can still change behavior for callers who relied on the old default path through the code.
- Side effects are implicit contracts. If a function has always logged, written to a file, or emitted an event, removing that side effect can break callers who depend on it even though it was never part of the explicit API.
- When refactoring, hold the behavioral boundary constant even if the implementation changes completely. Tests that verify behavior (not implementation) are the safety net here.
- For public APIs and shared libraries, backwards compatibility is almost always more important than internal cleanliness. Ugly-but-compatible beats clean-but-breaking.

See `examples/implicit-contract-break.md` for a case where a "safe" refactoring breaks callers through implicit behavioral changes.

### 6. Dependency Direction

**Question**: Does this change make the dependency graph simpler or more complex? Are dependencies flowing in the right direction?

**Key signals**:
- Dependencies should flow from unstable (frequently changing) code toward stable (rarely changing) code. If stable infrastructure code depends on volatile business logic, the direction is wrong.
- A new import or dependency crossing a package/module boundary is a stronger signal than one within the same module. Cross-boundary dependencies are harder to undo.
- Circular dependencies are a structural problem, not just a lint warning. They mean two modules cannot be understood, tested, or deployed independently.
- When tempted to add a dependency "just for one function," consider the transitive cost: you're also depending on everything that function depends on.
- If two modules keep needing to change together, they are coupled regardless of whether there is an explicit dependency between them. Consider whether they should be merged or whether a missing abstraction needs to be introduced between them.


## Scope Decision

After working through the relevant judgment dimensions, decide the scope of intervention:

- **Narrow local patch**: when the change is well-bounded, existing structure supports it cleanly, and no judgment dimension raised concerns.
- **Focused structural improvement**: when a purely local patch would deepen a problem identified by one or more judgment dimensions. Scope the improvement tightly to the concern raised — do not broaden into redesign.
- **Explicit pause**: when you don't yet understand why the code is shaped the way it is, or multiple judgment dimensions point in conflicting directions.

Use `references/risk-triage.md` when the scope decision is not clear from the judgment dimensions alone. See `examples/patch-vs-structural-improvement.md` for the classic scope decision case.


## Self-Correction Signals

Stop and revise when:
- you are implementing directly from the request without understanding the surrounding code
- you are treating the first editable location as the right implementation location
- you are using green tests or delivery pressure to justify a patch at a boundary you already suspect is wrong
- you are removing duplication without checking whether the duplicated code is actually diverging
- you are adding parameters or conditionals to an existing abstraction instead of questioning whether the abstraction is still right
- you are preserving an existing pattern without judging whether it is still the right one
- you are scattering error handling across layers without a clear owner
- you are expanding the work into redesign beyond what the change needs
- you cannot explain how the implementation supports both current intent and future maintainability


## Self-Check Before Exiting

- [ ] Did I work through the relevant judgment dimensions for this change?
- [ ] Did I follow all invariants (usage evidence over theoretical justification, question existing shapes)?
- [ ] Did I catch any self-correction signals?
- [ ] Does the implementation keep responsibilities clear and future changes easier?
- [ ] Am I exiting because the implementation is genuinely sound, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
