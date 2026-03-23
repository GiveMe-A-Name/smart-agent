# Lean Skill Patterns

## Purpose

Use this file when a skill draft is becoming long or repetitive.

The goal is not the shortest possible `SKILL.md`. The goal is a main file that stays complete at decision time and support folders that hold only the right kind of support material.

## Split Rule

Keep inline:
- trigger logic
- boundary choice
- constitution
- main workflow
- self-correction signals
- verification expectation

Move out of the main file by role:
- `references/` for detailed review prompts, long checklists, lookup tables, and extended eval notes
- `templates/` for reusable scaffolds or starter structures
- `examples/` for good, bad, or contrasting examples
- `scripts/` for deterministic helpers worth reusing

If moving a section would leave the main file unable to guide a first-pass decision, keep it inline.

## Folder Taxonomy

Use the narrowest folder that matches the file's job.

- `references/`: read for knowledge
- `templates/`: copy or adapt as a starting point
- `examples/`: compare against a standard
- `scripts/`: execute for deterministic work

Bad packaging often comes from collapsing all four jobs into `references/`.

## Redundancy Audit

Run this before finishing a skill, even if the user never mentioned length.

Ask:
1. Did I explain the same rule in more than one section?
2. Does this example add judgment, or just repeat the bullet?
3. Is this paragraph clarifying the law, or merely defending it?
4. Would a shorter cross-reference preserve the same value?

## Integration Before Accretion

When revising an existing skill, do not default to adding a new section just because the request names one topic.

Use this decision order:
1. merge into the strongest existing section
2. tighten existing wording
3. replace duplication with a cross-reference
4. add a new section only if the idea adds a new decision signal or workflow

The default move is integration. Appending is the last resort.

## Common Compression Moves

Use these moves in order:
1. merge duplicate sections
2. replace repeated explanation with one rule plus one example
3. move heavy support material to the correct support folder
4. delete defensive prose that adds no new decision signal

If a requested addition overlaps with existing overview, workflow, checklist, or examples, keep one home for the idea and strengthen that location instead of creating a parallel block.

## Signs The Draft Should Split

Split when:
- the core law is already clear but examples and checklists keep growing
- one section has turned into a lookup appendix
- multiple audiences or scenarios need different supporting detail
- the main file is repeating support material just to stay self-contained

Do not split when:
- the content is still core law
- the main workflow would become unclear without it
- the split only optimizes file length instead of decision quality

## Bad Move

"Put trigger rules, constitution, and the main workflow into support files so the main file stays short."

Why this fails:
- the main file becomes hollow
- the agent has to go hunting for the actual law
- shortness replaces usability

## Good Move

"Keep the law inline. Move the expanded examples, templates, review prompts, and deterministic helpers into the right support folders, then point to them clearly."

Why this works:
- the first-pass decision stays local
- support detail remains available
- the main file becomes shorter without becoming weaker
