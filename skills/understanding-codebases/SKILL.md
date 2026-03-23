---
name: understanding-codebases
description: Use when one or two targeted repository lookups are not enough and the agent must understand an unfamiliar repository slice, trace call flow, compare patterns, or prove ownership and likely change points across multiple files or layers before proposing changes.
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
- building a task-specific repository slice instead of summarizing the whole repo or doing generic preflight exploration
- locating entry points, key implementation files, and nearby tests when they materially support the question
- tracing one main call path from entry toward implementation
- finding nearby comparable implementations and extracting local patterns when available
- identifying likely change points and explaining why they are stronger candidates than nearby files when change placement is part of the question
- surfacing what is still unknown or only hypothesized

This skill does not own:
- clarifying product intent, scope, or acceptance criteria
- sequencing implementation work or writing a change plan
- recommending a verification strategy
- deciding whether implementation is complete

## Constitution

- Understand before suggesting.
- Prefer code evidence over intuition.
- Do not use this skill as a mandatory preflight before editing, answering, or clarifying.
- Learn from nearby patterns before inferring ownership, local conventions, or change points.
- Trace beyond export, registration, config, and facade layers when possible.
- Distinguish entry, orchestration, implementation, and shared utility layers.
- Do not infer ownership or the main change point from filenames, package names, or top-level exports alone.
- Explain why a candidate change point belongs here and not in nearby modules.
- Stop once the current ownership or change-point question is grounded; do not keep mapping the repo just because more context exists.
- Mark unproven claims as hypotheses or unknowns.
- Produce analysis, not implementation commitments.

## Core Moves

These are understanding moves, not a fixed sequence. Use the ones that help answer the current code-understanding question.

- Anchor on the current question: ownership, main call flow, candidate change point, local convention, or behavior source.
- Build a local repository slice around the files, layers, and tests that matter to that question.
- Find the strongest entry points and trace toward behavior-changing code instead of stopping at exports, config, or facades.
- Compare nearby implementations to learn local conventions before inferring how this slice works.
- Distinguish proven relationships from hypotheses, and keep track of what is still unknown.
- Stop once the current question is grounded well enough to answer safely.

## Grounded Understanding

You have enough understanding for the current question when, from code evidence, you can do most of the following that actually matter here:

- explain the relevant repository slice and how its parts relate
- trace enough of the call flow to answer the current question safely
- point to concrete evidence for ownership, behavior, conventions, or likely change placement when relevant
- name the strongest nearby patterns or explicitly note that comparable examples are sparse
- use nearby tests as evidence when they materially support the question, or say they were not useful or not present
- state the major unknowns that remain

If you still cannot answer the current question without leaning on intuition, keep investigating instead of sounding certain.

## Self-Correction Signals

Stop and revise when:

| Signal | What it means | What to do |
|---|---|---|
| You are suggesting code changes before building a repository slice | You are acting on intuition too early | Go back and map the local slice first |
| You only found export, registration, config, or facade layers | You have not reached real implementation yet | Keep tracing toward behavior-changing code |
| You listed files without explaining relationships | You searched, but did not model the code | Add call flow and responsibility boundaries |
| You did not inspect similar implementations when they exist | You are inventing patterns from scratch | Inspect nearby comparable examples first |
| You are treating a dependent or consumer module as the main change point without proof | Ownership is likely misidentified | Re-check which layer owns the behavior |
| You are stating guesses as facts | Evidence and inference are mixed together | Relabel claims as proven, hypothesis, or unknown |
| Your analysis is turning into implementation design | You are crossing into planning | Restate the boundary and return to analysis |
| You have no open unknowns in a genuinely unfamiliar area | You are likely overconfident | Re-examine what is still unproven |
| One or two targeted lookups would have answered the question | You escalated into deep analysis too early | Stop deep analysis and answer from the narrower evidence already available |
| You are still exploring after the ownership or change-point question is already grounded | Repository mapping became habitual instead of necessary | Stop and report the current analysis |
