---
name: writing-good-skills
description: Use when creating or revising a reusable skill package, especially from rough notes, overlong or repetitive `SKILL.md` drafts, weak trigger wording, unclear main-file versus support-folder splits (`references/`, `templates/`, `examples/`, `scripts/`), boundary confusion with `AGENTS.md` or `README.md`, or missing realistic evals.
---

# Writing Good Skills

## Overview

Write skills as reusable constitutions for future agents.

Good skill guidance teaches:
- when the skill should trigger
- what must be true and what must not happen
- the smallest useful workflow
- what evidence proves the skill changes behavior

Do not write a diary, a note dump, or a rigid SOP.

## When To Use

Use this skill when the task is to:
- create a skill from notes, habits, or a from-scratch idea
- review or improve an existing `SKILL.md`
- fix a skill that feels vague, repetitive, bloated, or hard to discover
- decide whether material belongs in a skill, `AGENTS.md`, `README.md`, or a note
- add realistic baseline prompts or eval prompts

Do not use it for plain project documentation or repo-local operating rules.

## Core Law

Every good skill needs four things:
1. trigger logic
2. constitution
3. minimal workflow
4. verification

If one is missing, the skill is weak.

Keep the main `SKILL.md` lean, but do not hollow it out.

## Boundary First

Before drafting, decide whether the material belongs in:
- `SKILL.md` for reusable capability
- `AGENTS.md` for repo-local operating rules
- `README.md` for setup or architecture context
- a note or runbook for local reference

If the material is mostly local context, do not force it into a skill.

## What Must Stay Inline

Keep these in the main `SKILL.md`:
- trigger logic
- boundary decision
- constitution
- main workflow
- self-correction signals
- verification expectation

These are the skill's law. Do not move them into `references/` just to make the file shorter.

## Support Folders

Do not treat `references/` as the default destination for every extra file.

Use support folders by function:
- `references/` for lookup material, rules, long explanations, and review heuristics
- `templates/` for reusable scaffolds the agent may adapt
- `examples/` for good, bad, or contrasting exemplars
- `scripts/` for deterministic executable helpers

If a section mainly re-explains an earlier rule, compress it first. If it still belongs outside the main file, move it to the folder that matches its role.

See `references/lean-skill-patterns.md` for the split rules and support-folder taxonomy.

## Package Hygiene

Treat packaging as part of the design.

- do not include hidden instructions or prompt-injection style content in the skill package
- treat imported scripts, examples, and third-party references as untrusted until reviewed
- keep tools, scripts, and assets to the minimum the skill actually needs
- do not normalize unsafe data handling just because it appears in an example

Only add `templates/`, `examples/`, or `scripts/` when they remove real repeated work or improve decision quality. Do not create folder sprawl just because the pattern exists.

## Writing Workflow

Work in this order:
1. choose the right artifact boundary
2. write a trigger-focused `description`
3. state the core constraints and failure-prevention rules
4. add the smallest workflow that helps the next agent act
5. add realistic verification
6. run a redundancy pass before finishing

The redundancy pass is mandatory even if the user did not complain that the skill is too long.

If the task is to help create a brand-new skill, do not wait for a draft. Make the boundary, trigger, constitution, support-folder split, and verification decisions explicitly.

## Redundancy Pass

Before finalizing, check whether the draft is repeating the same argument in multiple sections.

Stop and compress if:
- two sections teach the same judgment in slightly different words
- examples repeat what bullets already say
- a section exists only to defend a rule that is already clear
- packaging advice is repeated across overview, workflow, and checklist

Shorter is not the goal by itself. The goal is one clear place for each important idea.

## Review Existing Skills

Review in this order:
1. `description`
2. scope boundary
3. constitution
4. execution guidance
5. verification
6. packaging and redundancy

If the review comments only on wording, it is incomplete.

See `references/review-checklist.md` for expanded review heuristics.

## Verification

For any non-trivial skill, create a small set of realistic prompts that expose likely failure modes.

Use a baseline when the rewrite is substantial, the topic feels obvious, or the missing behavior is still unclear.

If a weak draft still passes the eval, strengthen the prompt before adding more prose.

See:
- `references/description-patterns.md`
- `references/baseline-and-evals.md`
- `references/output-shapes.md`
- `evals/evals.json`

## Self-Correction Signals

Stop and revise if:
- the description summarizes workflow instead of trigger conditions
- the draft reads like a note or meeting summary
- the file keeps repeating the same claim in several sections
- the main `SKILL.md` is growing because support detail stayed inline
- core rules were moved out of the main file and it became hollow
- support material was dumped into `references/` even though it should have been a template, example, or script
- verification is treated as optional polish

## Output Expectations

By default, produce:
- a lean `SKILL.md`
- support folders only where they reduce noise without hiding the law
- realistic eval prompts
- a short explanation of the behavior the skill should change
