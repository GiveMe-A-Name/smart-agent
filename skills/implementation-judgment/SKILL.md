---
name: implementation-judgment
description: Use before writing any code to decide how to land the change — not just where. Knowing the change point does not answer whether the approach is right. Invoke unless the work is purely mechanical with no behavioral or structural implications.
---

# Implementation Judgment

Implement changes with engineering judgment, not just task completion.

This skill applies engineering judgment to code changes — not just locating where to change, but reasoning about how the change should land given prior decisions, existing constraints, and long-term maintainability. The result may be a narrow patch, a focused structural improvement, or an explicit pause — but the value is in the judgment, not just the routing.

## Trigger Logic

**Invocation default**: The trigger state is simple: code is about to be written. The cost of an unnecessary invocation is a few extra reasoning steps. The cost of a missed invocation is a patch that weakens structure, moves responsibility into the wrong layer, or makes the next change harder — often invisibly. Invoke unless the work is purely mechanical with no behavioral effect.

**Knowing where to change is not the same as knowing how.** Having a plan, a root cause, or a change point identified answers the first question. Whether the approach is the right one — given tradeoffs, failure modes, and what the current code is protecting — is a separate question. Prior investigation, planning, or root-cause analysis does not substitute for this judgment — they ground it.

**Prerequisites**: This skill requires code evidence and clarified intent to reason responsibly. If code evidence is still missing, build that understanding first and return here. If intent is still unclear, clarify that first and return here. These are sequencing dependencies — not reasons to skip this skill, but work to complete before this skill can be useful.


## Boundary

This skill owns:
- reasoning from clarified intent and grounded code evidence to responsible implementation
- understanding why relevant code is shaped the way it is
- judging whether candidate change points and existing patterns should be preserved, extended, tightened, or improved so responsibilities stay coherent
- deciding whether a local patch is sufficient or a focused structural improvement is warranted
- balancing delivery speed, simplicity, system coherence, and future maintainability

This skill does not own:
- product clarification from scratch
- repository analysis as an end in itself — if code evidence is missing, that work belongs before this skill, not inside it
- generic task planning
- unrestricted refactoring
- final completion or release judgment


## Invariants

- Do not treat the task description as the full implementation truth.
- Do not assume existing code shape is arbitrary before considering what it may be protecting.
- Do not infer unconfirmed product intent from code shape alone.
- Do not let deadline or delivery pressure justify moving responsibility into the wrong layer.
- Prefer the least invasive change that keeps responsibilities clear and future changes easier.
- Do not treat theoretical justification as a substitute for usage evidence. "This supports testability", "this could be useful", "this is clean design" are arguments, not evidence. Evidence is: who calls this, who depends on this, what breaks if this is removed.
- **Before exiting this skill, you MUST complete the Self-Check section at the end**

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

## Self-Check Before Exiting

- [ ] Did I follow all invariants (understand surrounding code, base on usage evidence)?
- [ ] Did I catch any self-correction signals?
- [ ] Does the implementation keep responsibilities clear and future changes easier?
- [ ] Am I exiting because the implementation is genuinely sound, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
