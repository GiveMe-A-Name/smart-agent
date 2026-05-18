# Lean Skill Patterns

## Purpose

Use this file when a skill draft is becoming long or repetitive.

The goal is not the shortest possible `SKILL.md`. The goal is a main file that stays complete at decision time and support folders that hold only the right kind of support material.

Keep the package capability-first: the main file should teach principles and constraints, not hide skill-to-skill routing in support docs.

## Why Lean Matters Mechanically

Two mechanisms explain why a long main file reduces skill reliability, even when all the content is correct:

**Attention dilution**: As an agent generates more output tokens, it pays progressively less attention to earlier instructions in the context. A long skill file exacerbates this — the longer the file, the more likely that content from later sections is underweighted when the agent is executing.

**Lost in the middle**: In long-context settings, information placed in the middle of a file is attended to less than information at the beginning or end — even when the agent has demonstrably "read" it. Principle or constraint logic buried mid-file may be correctly parsed yet underweighted at the moment it matters.

These are not aesthetic concerns. A skill file long enough to have a real "middle" has a correctness problem, not just a style problem. This is why front-loading matters: `## Principles` and `## Constraints` belong near the top, not after several paragraphs of supporting material.

**Constraint density as a distinct risk**: file length is not the only overload failure. A short file can still underperform if a single section packs many independent constraints without a clear priority ordering. When constraints compete for attention without ranking, agents tend to satisfy some and silently drop others. The fix is restructuring around a unifying principle, not just shortening.

## Split Rule

Keep inline:
- frontmatter `description` for pre-load skill discovery
- `## Principles`
- `## Constraints`

Move out of the main file by role:
- `references/` for detailed review prompts, long checklists, and lookup tables
- `templates/` for reusable scaffolds or starter structures
- `examples/` for good, bad, or contrasting examples
- `scripts/` for deterministic helpers worth reusing

If moving material would leave the main file unable to guide first-pass execution judgment, keep it inline.

If moving material would push principle or constraint logic into support files, keep it inline.

## Integration Before Accretion

When revising an existing skill, do not default to adding a new section or parallel explanation just because the request names one topic.

Use this decision order:
1. merge the idea into `## Principles` or `## Constraints`
2. tighten existing wording
3. replace repeated explanation with one rule plus one example
4. move heavy support material to the correct support folder
5. delete defensive prose that does not strengthen a principle or constraint
6. replace duplication with a cross-reference
7. add a new section only if the idea cannot fit into `## Principles`, `## Constraints`, or a support file

The default move is integration. Appending is the last resort.

If a requested addition overlaps with existing overview, workflow, checklist, or examples, keep one home for the idea and strengthen that location instead of creating a parallel block.

## Good Move

"Keep principles and constraints inline. Move the expanded examples, templates, review prompts, and deterministic helpers into the right support folders, then point to them clearly."

"Keep capability-first principles and constraints in the main file. Use support docs to deepen the current skill, not to replace constraint clarity with a hidden routing graph."

"Use action hints only when they improve judgment. Do not turn them into mandatory SOP or output-template rules."

Why this works:
- first-pass execution judgment stays local
- support detail remains available
- the main file becomes shorter without becoming weaker
- support docs reinforce the same principles and constraints instead of smuggling in skill-to-skill routing
- the package teaches flexible judgment instead of format compliance
