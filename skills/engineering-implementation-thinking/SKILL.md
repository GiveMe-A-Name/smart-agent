---
name: engineering-implementation-thinking
description: Use when implementing or modifying code requires engineering judgment about intent, existing architecture, historical code shape, hidden constraints, and long-term maintainability.
---

# Engineering Implementation Thinking

Implement changes with engineering judgment, not just task completion.

This skill helps the agent decide how to land code responsibly in an existing system: understand the real intent, read current code as evidence of prior decisions, respect meaningful constraints, and avoid both shallow patching and unnecessary redesign.

## Trigger Logic

Use this skill when:
- a non-trivial change is being implemented in an existing codebase
- understanding why current code is shaped this way may change how the work should be done
- a narrow patch could complete the task, but might weaken structure or maintainability
- architecture boundaries, contracts, compatibility concerns, or historical decisions materially affect the implementation

Do not use this skill when:
- the change is tiny, local, and has one obvious low-risk path
- the main missing step is still requirement clarification or repository exploration
- the work is purely mechanical, clerical, or stylistic
- the main question is whether work is complete rather than how it should be implemented

## Boundary

This skill owns:
- reasoning from requested intent to responsible implementation
- understanding why relevant code is shaped the way it is
- judging where the change should land so responsibilities stay coherent
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

This skill is not the right tool yet when the main missing piece is still:
- understanding what this part of the codebase does, how behavior flows, or who owns the touched area
- clarifying what the user really wants or which decisions still require human intent

Use this skill when the remaining question is:
- given the real intent and the code evidence, what is the right way to implement this here?

## Invariants

- Do not treat the task description as the full implementation truth.
- Do not assume the smallest edit is the best edit.
- Do not assume existing code shape is arbitrary before considering what it may be protecting.
- Do not preserve a weak pattern blindly just because it already exists.
- Do not use maintainability as an excuse for unnecessary redesign.
- Prefer the least invasive change that keeps responsibilities clear and future changes easier.

## Core Questions

A strong result can answer:
- What is the real intent of this change?
- Why is the current code shaped this way?
- Which current patterns are essential, accidental, or ready for focused improvement?
- Where should this change live so ownership stays clear?
- Would the smallest patch make future work harder?
- Should this change preserve, extend, or tighten the existing structure?

## Self-Correction Signals

Stop and revise when:
- you are implementing directly from the request without understanding the surrounding code
- you are treating the first editable location as the right implementation location
- you are making the narrowest patch even though it adds duplication, coupling, or confusion
- you are preserving an existing pattern without judging whether it is still the right one
- you are expanding the work into redesign beyond what the change needs
- you cannot explain how the implementation supports both current intent and future maintainability

## What Good Looks Like

A good result:
- fits the real structure of the codebase
- reflects understanding of why the touched code exists in its current form
- preserves or intentionally improves boundaries
- avoids both thoughtless minimal patching and unnecessary redesign
- makes related future changes easier, clearer, or safer

Use `references/risk-triage.md` when you need help deciding whether the change needs a narrow patch, a focused structural improvement, or an explicit pause due to weak understanding.
