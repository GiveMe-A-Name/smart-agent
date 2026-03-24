---
name: engineering-implementation-thinking
description: Use when implementing or modifying code requires engineering judgment about clarified intent, architecture, historical code shape, hidden constraints, and long-term maintainability, and the needed code evidence is already available or can be gathered with focused investigation.
---

# Engineering Implementation Thinking

Implement changes with engineering judgment, not just task completion.

This skill helps the agent decide how to land code responsibly in an existing system: work from clarified intent, read current code as evidence of prior decisions, respect meaningful constraints, and choose among a narrow patch, a focused structural improvement, or an explicit pause when understanding is still too weak.

## Trigger Logic

Use this skill when:
- the requested intent is already clarified enough, or can be clarified enough, to support responsible implementation judgment
- the relevant code evidence is already grounded enough, or can be gathered with focused investigation, to support implementation judgment
- a non-trivial change is being implemented in an existing codebase
- understanding why current code is shaped this way may change how the work should be done
- a narrow patch could complete the task, but might weaken structure or maintainability
- architecture boundaries, contracts, compatibility concerns, or historical decisions materially affect the implementation

Do not use this skill when:
- the change is tiny, local, and has one obvious low-risk path
- the main missing step is still requirement clarification or repository exploration
- the work is purely mechanical, clerical, or stylistic
- the main question is whether work is complete rather than how it should be implemented
- the agent is using engineering judgment as a default warm-up for every non-trivial implementation task

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
- Why is the current code shaped this way?
- Which current patterns are essential, accidental, or ready for focused improvement?
- Which candidate change points actually support coherent ownership?
- Would the smallest patch make future work harder?
- Should this change be a local patch, a focused structural improvement, or an explicit pause pending better understanding?

Use `references/risk-triage.md` when these questions do not resolve the patch-vs-improvement-vs-pause choice. See `examples/patch-vs-structural-improvement.md` for a contrast pair showing how the same request can warrant either a narrow patch or a focused structural improvement depending on what the current code shape reveals.

## Self-Correction Signals

Stop and revise when:
- you are implementing directly from the request without understanding the surrounding code
- you are treating the first editable location as the right implementation location
- you are using green tests or delivery pressure to justify a patch at a boundary you already suspect is wrong
- you are making the narrowest patch even though it adds duplication, coupling, or confusion
- you are preserving an existing pattern without judging whether it is still the right one
- you are expanding the work into redesign beyond what the change needs
- you cannot explain how the implementation supports both current intent and future maintainability

