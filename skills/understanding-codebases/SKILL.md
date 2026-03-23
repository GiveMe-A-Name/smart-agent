---
name: understanding-codebases
description: Use when one or two targeted lookups are not enough and the agent must understand a repository slice across multiple files or layers to trace behavior, compare patterns, or identify ownership and candidate change points safely.
---

# Codebase Understanding

Build grounded understanding of the code before suggesting implementation.

Use this skill to learn the local structure of a repository, identify the files
and modules that matter, trace how behavior flows through the code, and explain
where a change likely belongs. This skill analyzes code structure; it does not
clarify product requirements, create an implementation plan, or decide whether
work is complete.

## Trigger Logic

Use this skill when:
- one or two targeted repository lookups are not enough to answer the current question safely
- the task requires repository-slice analysis across multiple files or layers
- the agent must trace a main call flow, compare local patterns, or prove ownership before suggesting where work belongs
- unfamiliarity with the local slice makes confident action unsafe without deeper code evidence

Do not use this skill when:
- a single file, obvious edit path, or one or two targeted lookups already answer the question
- the task is a shallow lookup about terminology, existing flags, exposed configuration, exported options, or already-obvious behavior
- the goal is only to decide whether the request is clear enough to continue clarification
- the agent is using repository understanding as a default warm-up before every action

## Boundary

This skill owns:
- building a task-specific repository slice for the current question
- tracing enough call flow to ground ownership, behavior, or candidate change points
- comparing nearby implementations when local patterns matter
- stating what is proven, hypothesized, and still unknown

This skill does not own:
- clarifying product intent, scope, or acceptance criteria
- writing a change plan or sequencing implementation work
- final engineering judgment about how a change should be implemented once repository evidence is understood
- recommending verification strategy or completion judgment

## Constitution

- Understand before suggesting.
- Prefer code evidence over intuition.
- Do not use this skill as a default warm-up before every action.
- Learn from nearby patterns before inferring ownership or conventions.
- Trace past export, registration, config, and facade layers when possible.
- Mark unproven claims as hypotheses or unknowns.
- Produce analysis, not implementation commitments.

## Core Moves

These are understanding moves, not a fixed sequence. Use the ones that help answer the current code-understanding question.

- Anchor on the current question.
- Build the smallest repository slice that can answer it safely.
- Trace toward behavior-changing code instead of stopping at facades.
- Compare nearby patterns when they materially inform the answer.
- Stop once the answer is grounded well enough to avoid intuition.

## Grounded Understanding

You have enough understanding for the current question when, from code evidence, you can do the relevant subset of the following:

- explain the repository slice that matters to this question
- trace enough call flow to answer safely
- point to concrete evidence for ownership, behavior, conventions, or candidate change points when relevant
- state the major unknowns that remain

If you still need intuition to bridge the answer, keep investigating.

## Self-Correction Signals

Stop and revise when:

- you are suggesting changes before building a local repository slice
- you stopped at export, registration, config, or facade layers
- you listed files without explaining relationships or call flow
- you are treating guesses about ownership or change placement as facts
- you are drifting into planning or still exploring after the question is already grounded
