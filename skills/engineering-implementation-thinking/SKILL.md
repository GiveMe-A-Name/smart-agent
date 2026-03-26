---
name: engineering-implementation-thinking
description: Use before writing any code to decide how to land the change — not just where. Knowing the change point does not answer whether the approach is right. Invoke unless the work is purely mechanical with no behavioral or structural implications. Do not use when code evidence is still too thin to reason about change points — gather that evidence first.
---

# Engineering Implementation Thinking

Implement changes with engineering judgment, not just task completion.

This skill helps the agent decide how to land code responsibly in an existing system: work from clarified intent, read current code as evidence of prior decisions, respect meaningful constraints, and choose among a narrow patch, a focused structural improvement, or an explicit pause when understanding is still too weak.

## Trigger Logic

**Invocation default**: The cost of an unnecessary invocation is a few extra reasoning steps. The cost of a missed invocation is a patch that weakens structure, moves responsibility into the wrong layer, or makes the next change harder — often invisibly. Invoke this skill before writing code unless the work is confirmed to be purely mechanical with no structural judgment involved.

**Knowing where to change is not the same as knowing how.** Identifying the change point (which file, which line) answers the first question. Whether the approach is the right one — given tradeoffs, failure modes, and what the current code is protecting — is a separate question that this skill exists to answer. Do not skip this skill on the grounds that the change point is already clear.

Use this skill when:
- a non-trivial change is being implemented in an existing codebase
- clarified intent and enough code evidence are already available to decide how to land the change responsibly
- the main question is whether to make a local patch, preserve or tighten a boundary, or do a focused structural improvement instead
- understanding why the current code is shaped this way may change how the work should be done
- architecture boundaries, contracts, compatibility concerns, or historical decisions materially affect the implementation
- the quickest edit could complete the task, but might weaken structure or maintainability

Do not use this skill when:
- the work is purely mechanical, clerical, or stylistic with no structural judgment involved — fixing a typo, updating a string literal, renaming with no behavioral effect
- the main question is whether work is complete rather than how it should be implemented

If code evidence is still too thin to reason about change points, build that understanding first. If intent is still unclear, resolve that first. Return to this skill once both are grounded.

## Boundary

This skill owns:
- reasoning from clarified intent and grounded code evidence to responsible implementation
- understanding why relevant code is shaped the way it is
- judging whether candidate change points and existing patterns should be preserved, extended, tightened, or improved so responsibilities stay coherent
- deciding whether a local patch is sufficient or a focused structural improvement is warranted
- balancing delivery speed, simplicity, system coherence, and future maintainability

This skill does not own:
- product clarification from scratch
- broad repo exploration as the final output
- generic task planning
- unrestricted refactoring
- final completion or release judgment

## Scope Edge

This skill assumes the agent can obtain enough code evidence to reason responsibly, but it does not own repository analysis as an end in itself.

If implementation judgment is blocked by missing code evidence, gather the missing evidence first. If that requires deeper repository-slice analysis across multiple files or layers, do that first before returning to this skill.

This skill is not the right tool yet when the main missing piece is still:
- understanding what this part of the codebase does, how behavior flows, or who owns the touched area
- clarifying what the user really wants or which decisions still require human intent

Use this skill when the remaining question is:
- given the clarified intent and grounded code evidence, should this be a local patch, a focused structural improvement, or an explicit pause?

## Invariants

- Do not treat the task description as the full implementation truth.
- Do not assume existing code shape is arbitrary before considering what it may be protecting.
- Do not infer unconfirmed product intent from code shape alone.
- Do not let deadline or delivery pressure justify moving responsibility into the wrong layer.
- Prefer the least invasive change that keeps responsibilities clear and future changes easier.

## Core Questions

A strong result can answer:
- What clarified implementation-relevant intent must this change serve?
- Does the shape of the problem itself — multiple similar issues, repeated patterns, or scattered ownership — signal a missing abstraction rather than N independent fixes?
- Why is the current code shaped this way?
- Which current patterns are essential, accidental, or ready for focused improvement?
- Which candidate change points actually support coherent ownership?
- Would the smallest patch make future work harder?
- Should this change be a local patch, a focused structural improvement, or an explicit pause pending better understanding?

Use `references/risk-triage.md` when these questions do not resolve the patch-vs-improvement-vs-pause choice. See `examples/patch-vs-structural-improvement.md` for a general contrast pair, and `examples/security-review-duplicate-helper.md` for a review-remediation case where fixing the issue responsibly requires re-centering a shared helper instead of scattering local copies.

## Self-Correction Signals

Stop and revise when:
- you are implementing directly from the request without understanding the surrounding code
- you are treating the first editable location as the right implementation location
- you are using green tests or delivery pressure to justify a patch at a boundary you already suspect is wrong
- you are making the narrowest patch even though it adds duplication, coupling, or confusion
- you are preserving an existing pattern without judging whether it is still the right one
- you are expanding the work into redesign beyond what the change needs
- you cannot explain how the implementation supports both current intent and future maintainability
