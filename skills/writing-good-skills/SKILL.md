---
name: writing-good-skills
description: Use when creating or revising a reusable skill package, especially from rough notes, overlong or repetitive `SKILL.md` drafts, weak trigger wording, unclear main-file versus support-folder splits (`references/`, `templates/`, `examples/`, `scripts/`), boundary confusion with `AGENTS.md` or `README.md`, or missing realistic evals.
---

# Writing Good Skills

## Overview

Write skills as reusable constitutions for future agents.

Good skill guidance teaches:
- when the skill should trigger
- what capability the skill owns and where its boundary is
- what must be true and what must not happen
- the smallest useful workflow
- what evidence proves the skill changes behavior

Do not write a diary, a note dump, a rigid SOP, or a hidden skill-to-skill routing graph.

## When To Use

Use this skill when the task is to:
- create a skill from notes, habits, or a from-scratch idea
- review or improve an existing `SKILL.md`
- fix a skill that feels vague, repetitive, bloated, or hard to discover
- decide whether material belongs in a skill, `AGENTS.md`, `README.md`, or a note
- add realistic baseline prompts or eval prompts

Do not use it for plain project documentation or repo-local operating rules.

## Core Law

Every good skill needs six things:
1. trigger logic
2. boundary
3. constitution
4. minimal workflow
5. self-correction signals
6. verification

If one is missing, the skill is weak.

Keep the main `SKILL.md` lean, but do not hollow it out.

A good skill is capability-first: it teaches judgment, not orchestration.

Use the skill to clarify:
- when the capability applies
- what the capability owns and does not own
- what must be true before acting
- what must not happen inside the skill boundary

Do not use the skill to pre-write the agent's next move with skill-to-skill routing such as handoff chains, named-skill routing graphs, or mandatory "use X next" instructions.

When a skill reaches the edge of its scope, state what is missing, what the skill does not own, and why continuing inside the skill would mislead. Let the agent choose the next action from the capability map.

When revising an existing skill, default to integration over accretion.
Do not append local prose until you have checked whether the same idea already has a better home in the current `SKILL.md`.

## Boundary First, Not Routing

Before drafting, decide whether the material belongs in:
- `SKILL.md` for reusable capability
- `AGENTS.md` for repo-local operating rules
- `README.md` for setup or architecture context
- a note or runbook for local reference

If the material is mostly local context, do not force it into a skill.

Prefer boundary language such as:
- `Do not use this skill when ...`
- `This skill does not own ...`
- `If this grounding is still missing, state it explicitly`
- `This request is outside this skill's scope`

Avoid routing language such as:
- `Route to ...`
- `Route back to ...`
- `Hand off to ...`
- `Use X skill next`

The difference matters: boundary language teaches scope judgment, while routing language teaches skill-to-skill routing.

If all named skill references were removed, the file should still teach a clear, useful capability. Use named skill references only when they prevent a severe category error.

A short relationship note can be acceptable when it clarifies capability boundaries without prescribing sequence. If a `Relationship to ...` section turns into handoff steps, neighboring-skill routing, or mandatory next actions, rewrite it as boundary language or remove it.

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

## Core Drafting Decisions

When writing or revising a skill, make these decisions explicit:
- artifact boundary
- trigger wording
- core constraints and failure-prevention rules
- the smallest workflow that helps the next agent act
- whether any wording is teaching boundary judgment or secretly encoding skill-to-skill routing
- verification and eval shape
- redundancy and support-folder split

The order is flexible. The goal is not to follow a ritual, but to ensure these decisions do not stay implicit.

The redundancy pass is mandatory even if the user did not complain that the skill is too long.

If the task is to help create a brand-new skill, do not wait for a draft. Make the boundary, trigger, constitution, support-folder split, and verification decisions explicitly.

## Integration Before Accretion

Treat this as a law, not a drafting preference.

Before adding new prose:
1. read the whole main `SKILL.md`
2. identify sections that already carry part of the same idea
3. strengthen the best existing home for the idea before creating a new one

A skill should have one clear home for each important idea. If the same judgment appears in overview, workflow, checklist, and examples, keep the strongest version and merge or delete the rest. See `references/lean-skill-patterns.md` for the decision order.

## Redundancy Pass

Before finalizing, check whether the draft is repeating the same argument in multiple sections.

Stop and compress if:
- two sections teach the same judgment in slightly different words
- examples repeat what bullets already say
- a section exists only to defend a rule that is already clear
- packaging advice is repeated across overview, workflow, and checklist

Shorter is not the goal by itself. The goal is clearer guidance with less repetition.

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

Good evals reward boundary recognition, explicit missing grounding, correct output shape, and refusal to fabricate what the skill cannot supply.

Do not reward naming the next skill, explicit handoff language, or acting as though good performance depends on skill-to-skill routing.

See:
- `references/description-patterns.md`
- `references/baseline-and-evals.md`
- `references/output-shapes.md`

## Self-Correction Signals

Stop and revise if:
- the description summarizes workflow instead of trigger conditions
- the draft reads like a note or meeting summary
- the file keeps repeating the same claim in several sections
- a local fix made the file longer without improving the skill's structure
- the draft relies on named skill references to explain its boundary
- the skill stops making sense once named skill references are removed
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
