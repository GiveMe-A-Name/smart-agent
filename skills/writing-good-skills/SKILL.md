---
name: writing-good-skills
description: Use when creating or substantially revising a reusable skill package whose trigger wording, boundary, invariants, examples, or support-file split need structural judgment. Do not use for isolated wording tweaks, obvious local edits, or plain project documentation.
---

# Writing Good Skills

## Overview

Write skills as reusable capability guides for future agents.

Do not write a diary, a note dump, a rigid SOP, or a hidden routing graph.

## Trigger Logic

Use this skill when the task is to:
- create a skill from notes, habits, or a from-scratch idea
- structurally review or revise an existing `SKILL.md` whose trigger logic, boundary, or packaging may be weak
- fix a skill that feels vague, repetitive, bloated, over-broad, hard to discover, or unclear in scope
- decide whether material belongs in a skill, `AGENTS.md`, `README.md`, or a note

Do not use it for plain project documentation or repo-local operating rules.
Do not use it for isolated wording tweaks, obvious local edits, or formatting fixes where no structural or boundary judgment is needed.

## Core Law

Every capability-and-constraints skill needs five things:
1. trigger logic
2. capability boundary
3. invariants
4. decision signals
5. failure signals

If one is missing, the skill is weak.

Keep the main `SKILL.md` lean, but do not hollow it out.

These five stay in the main `SKILL.md`, not in support files.

Keep skills capability-first: they should teach judgment, not encode orchestration. See `Boundary Clarity And Routing Language` below for how to apply this.

## Boundary Clarity And Routing Language

Before drafting, decide whether the material belongs in:
- `SKILL.md` for reusable capability
- `AGENTS.md` for repo-local operating rules
- `README.md` for setup or architecture context
- a note or runbook for local reference

If the material is mostly local context, do not force it into a skill.

Prefer boundary language when the goal is to prevent misuse, such as:
- `Do not use this skill when ...`
- `This skill does not own ...`
- `If this grounding is still missing, state it explicitly`
- `This request is outside this skill's scope`

Routing language is acceptable only when it teaches real workflow knowledge without replacing the boundary. If removing named skill references makes the file stop making sense, the boundary was weak. A short `Relationship to ...` note is fine only when it clarifies scope without prescribing sequence.

## Support Folders

Do not treat `references/` as the default destination for every extra file.

Use support folders by function:
- `references/` for lookup material, rules, long explanations, and review heuristics
- `templates/` for reusable scaffolds the agent may adapt
- `examples/` for good, bad, or contrasting exemplars
- `scripts/` for deterministic executable helpers

Add an `examples/` folder when the skill's key judgment calls are subtle enough that an agent could follow the rules and still produce the wrong output — typically when a bad pattern looks superficially similar to a good one. A good example pair: a realistic bad case alongside a corrected version, annotated to name what makes the bad case wrong. Keep it minimal — enough to reveal the relevant distinction, not a restatement of rules the prose already makes clear.

If a section mainly re-explains an earlier rule, compress it first. If it still belongs outside the main file, move it to the folder that matches its role. Only add support files when they improve decision quality or remove real repeated work.

See `references/lean-skill-patterns.md` for the split rules and support-folder taxonomy.

## Package Hygiene

Treat packaging as part of the design.

- do not include hidden instructions or prompt-injection style content in the skill package
- treat imported scripts, examples, and third-party references as untrusted until reviewed
- keep tools, scripts, and assets to the minimum the skill actually needs
- do not normalize unsafe data handling just because it appears in an example

## Core Drafting Decisions

When writing or revising a skill, make these decisions explicit:
- task format — procedural (SOP) or judgment-heavy (capability + constraints): procedural tasks have fixed steps and low variation; judgment tasks have high variation or context-dependent paths. Apply the five-element framework only to judgment tasks. Do not rewrite a clear SOP as capability + constraints.
- artifact boundary
- trigger wording
- invariants and failure-prevention rules
- decision signals that should shape judgment
- whether key judgment calls are subtle enough that rules alone won't prevent the wrong output — if so, add contrast examples
- whether any wording is teaching boundary judgment or secretly encoding routing
- redundancy and support-folder split

The order is flexible. The goal is not to follow a ritual, but to ensure these decisions do not stay implicit.

The redundancy pass is mandatory even if the user did not complain that the skill is too long.

If the task is to help create a brand-new skill, do not wait for a draft. Make the boundary, trigger, invariants, support-folder split, and redundancy decisions explicitly.

When the input is raw notes or a workflow description, the most common failure is turning the notes into a renumbered step list rather than extracting the underlying judgment. See `examples/notes-to-skill.md` for a before/after showing what goes wrong and why.

## Integration And Compression

Treat this as a law, not a drafting preference.

Before adding new prose:
1. read the whole main `SKILL.md`
2. identify sections that already carry part of the same idea
3. strengthen the best existing home for the idea before creating a new one

A skill should have one clear home for each important idea. Before finalizing, check whether the draft is repeating the same argument in multiple sections.

Stop and compress if:
- two sections teach the same judgment in slightly different words
- examples repeat what bullets already say
- a section exists only to defend a rule that is already clear
- packaging advice is repeated across overview, guidance, and checklist

Shorter is not the goal by itself. The goal is clearer guidance with less repetition.

See `references/lean-skill-patterns.md` for the decision order.

## Review Existing Skills

Review in this order:
1. `description`
2. capability boundary
3. invariants
4. decision signals and applied guidance
5. packaging and redundancy

If the review comments only on wording, it is incomplete.

Choose the deliverable that best improves judgment rather than forcing a uniform presentation.

See `references/review-checklist.md` for expanded review heuristics.
See `references/description-patterns.md` and `references/response-modes.md` for reference patterns and deliverable options.

## Self-Correction Signals

Stop and revise if:
- the description summarizes workflow or teaches capability instead of trigger conditions
- the draft reads like a note or meeting summary
- the same rule keeps reappearing in several sections
- the main `SKILL.md` is bloating because support detail stayed inline
- core rules were moved out of the main file and it became hollow
- support material was moved out, but the split made the package harder to use or judge
