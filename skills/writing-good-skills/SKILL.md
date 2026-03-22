---
name: writing-good-skills
description: Use when creating, reviewing, or improving a `SKILL.md`-based skill, especially when turning notes into a reusable skill, fixing vague trigger logic, deciding whether content belongs in a skill versus `AGENTS.md` or `README.md`, or adding verification prompts.
---

# Writing Good Skills

## Overview

Write a skill as a reusable constitution for future agents, not as a diary, a note dump, or a rigid SOP.

The goal is to encode the judgment agents usually skip:
- when the skill should trigger
- what constraints matter
- what failure signals require rethinking
- what evidence proves the skill changes behavior

## When To Use

Use this skill when the user asks to:
- create a skill from notes, habits, or a proven workflow
- improve or review an existing `SKILL.md`
- fix a skill that feels vague, broad, or hard to discover
- decide whether content belongs in a skill versus `AGENTS.md`, `README.md`, or a note
- add baseline prompts, eval prompts, or a verification loop

Do not use this skill for:
- plain project documentation
- repo-specific operating rules
- one-off notes that are not meant to become reusable agent capability

## Scope Boundary

A skill should teach reusable capability, not preserve local context.

Before drafting, decide whether the material belongs in:
- `SKILL.md` for reusable behavior across future conversations
- `AGENTS.md` for repo-local operating rules
- `README.md` for project setup or architecture context
- a note or runbook for local reference only

If the artifact is mainly local context rather than reusable capability, stop and write the right document instead.

## Core Pattern

A strong skill has four layers:
1. Triggering: when it should load
2. Constitution: what must be true, what must not happen
3. Execution guidance: the smallest useful workflow
4. Verification: prompts or checks that prove behavior changed

If one layer is missing, the skill is usually weak:
- no triggering -> hard to discover
- no constitution -> becomes a shallow checklist
- no execution guidance -> sounds right but is hard to use
- no verification -> cannot prove it changed behavior

## Constitution

A good skill must:
- produce a reusable skill, not a broad note dump
- choose skill versus `AGENTS.md` / `README.md` / note before drafting
- use `description` for triggering, not workflow summary
- keep `SKILL.md` lean and move only heavy supporting detail to `references/`
- teach judgment, not just formatting
- include realistic verification for non-trivial skills
- state what behavior it is supposed to change

Do not move core trigger logic, constitution, or the main workflow into `references/`.

## Invariants And Flexibility

Treat this skill in two zones.

Invariants:
- decide skill versus `AGENTS.md` / `README.md` / note before drafting
- keep `description` focused on trigger conditions
- make core constraints explicit
- require verification for non-trivial skills
- keep core trigger logic, constitution, and the main workflow in the main `SKILL.md`

Flexible areas:
- how much detail the review needs
- which examples best expose the likely failure mode
- how many eval prompts are enough for the case
- whether supporting detail should stay inline or move to `references/`

Do not turn flexible areas into rigid SOP.
Do not treat invariants as optional suggestions.

## Safety And Packaging Hygiene

Treat skill packaging as part of the design, not an afterthought.

When creating or reviewing a skill package:
- do not include hidden instructions, disguised tool calls, or prompt-injection style content in `SKILL.md`, `references/`, or examples
- treat `scripts/`, imported snippets, and third-party reference material as untrusted until reviewed
- give the skill only the minimum tools, scripts, and assets it actually needs
- do not normalize unsafe data handling patterns just because they appear in example code

If supporting files add execution risk without clear capability value, remove them or keep the skill documentation-only.

## Review Existing Skills

When reviewing an existing `SKILL.md`, check it in this order:
1. `description`: does it describe when to use the skill rather than the workflow?
2. scope: is it really a reusable skill?
3. constitution: are the key rules and failure modes explicit?
4. execution: does it guide action, not just state beliefs?
5. verification: are there realistic evals or equivalent checks?
6. packaging: is `references/` used only for heavy supporting material?

If the review skips trigger quality or verification, it is incomplete.

Detailed review prompts and report shapes live in `references/review-checklist.md` and `references/output-shapes.md`.

## Self-Correction Signals

Stop and revise if:
- the description starts summarizing workflow instead of trigger conditions
- the draft reads like a note or meeting summary
- the skill explains what to do but not what must not happen
- the examples are generic and not tied to real failure modes
- the draft has no realistic verification
- essential rules were moved into `references/` and the main file became hollow

## Recovery Loop

If verification stops teaching you anything, do not keep polishing the same draft.

- if the baseline already passes comfortably, strengthen the prompt or pick a failure mode with more discrimination
- if a weak draft still passes the eval, redesign the eval before rewriting more prose
- if two revision cycles do not improve behavior, revisit whether the missing constraint is still implicit
- if the material keeps collapsing into local context, stop and move it to `AGENTS.md`, `README.md`, or a note instead of forcing a skill

## Verification

For any non-trivial skill, create a small set of realistic prompts that expose likely failure modes.

Verification is part of authorship, not optional polish.

Use baseline prompts to show current failure behavior when creating or heavily revising a non-trivial skill. Use eval prompts to prove the finished skill changes that behavior.

See:
- `references/description-patterns.md` for trigger-focused description guidance
- `references/baseline-and-evals.md` for how to design baseline and eval prompts
- `evals/evals.json` for structured examples

## Quick Reference

- `description` -> when the skill should trigger
- `SKILL.md` -> core constraints and workflow
- `references/` -> heavy supporting detail, templates, expanded examples
- `evals/` -> realistic prompts that prove behavior change

## Common Mistakes

- writing a note and calling it a skill
- putting workflow summary into `description`
- writing a rigid SOP instead of constraints
- skipping verification because the topic feels obvious
- forcing repo-local material into a reusable skill
- moving essential rules into `references/`

## Output Expectations

When creating or improving a skill, produce:
- a lean `SKILL.md` with trigger, boundary, constitution, workflow, and verification expectations
- `references/` files only when they reduce noise without hiding core decisions
- a small set of realistic eval prompts
- a brief explanation of what behavior the skill is supposed to change
