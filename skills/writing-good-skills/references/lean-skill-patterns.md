# Lean Skill Patterns

## Purpose

Use this file when a skill draft is becoming long or repetitive.

The goal is not the shortest possible `SKILL.md`. The goal is a main file that stays complete at decision time and support folders that hold only the right kind of support material.

Keep the package capability-first: the main file should teach boundary judgment, not hide skill-to-skill routing in support docs.

## Why Lean Matters Mechanically

Two failure modes explain why a long main file reduces skill reliability, even when all the content is correct:

**Attention dilution**: As an agent generates more output tokens, it pays progressively less attention to earlier instructions in the context. A long skill file exacerbates this — the longer the file, the more likely that content from later sections is underweighted when the agent is executing.

**Lost in the middle**: In long-context settings, information placed in the middle of a file is attended to less than information at the beginning or end — even when the agent has demonstrably "read" it. Core law buried mid-file may be correctly parsed yet underweighted at the moment it matters.

These are not aesthetic concerns. A skill file long enough to have a real "middle" has a correctness problem, not just a style problem. This is why front-loading matters: trigger conditions and invariants belong near the top, not after several paragraphs of supporting material.

**Constraint density as a distinct risk**: file length is not the only overload failure. A short file can still underperform if a single section packs many independent constraints without a clear priority ordering. When constraints compete for attention without ranking, agents tend to satisfy some and silently drop others. The fix is restructuring around a unifying principle, not just shortening.

## Split Rule

Keep inline:
- trigger logic
- capability boundary
- invariants
- decision signals
- failure signals

Move out of the main file by role:
- `references/` for detailed review prompts, long checklists, and lookup tables
- `templates/` for reusable scaffolds or starter structures
- `examples/` for good, bad, or contrasting examples
- `scripts/` for deterministic helpers worth reusing

If moving a section would leave the main file unable to guide a first-pass decision, keep it inline.

If moving a section would push boundary language, core law, or capability-first judgment into support files, keep it inline.

## Folder Taxonomy

Use the narrowest folder that matches the file's job.

- `references/`: read for knowledge
- `templates/`: copy or adapt as a starting point
- `examples/`: compare against a standard — add when a bad pattern looks superficially similar to a good one and rules alone won't prevent the wrong output; a good example pair is a realistic bad case alongside a corrected version, annotated to name what makes the bad case wrong
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
4. add a new section only if the idea adds a new decision signal, invariant, or failure signal

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
- a single section lists many independent constraints without a unifying principle — restructure around a common principle before adding more

Do not split when:
- the content is still core law
- the main decision cues would become unclear without it
- the split only optimizes file length instead of decision quality

## Bad Move

"Bury trigger conditions or core invariants in the middle of a long main file where they will be read but underweighted."

"Put trigger rules, invariants, and decision cues into support files so the main file stays short."

"Use support docs mainly to explain which named skill should run next instead of clarifying the current skill's boundary."

"Make every response follow the same output shape so the skill feels consistent."

Why this fails:
- the main file becomes hollow
- the agent has to go hunting for the actual law
- shortness replaces usability
- support docs quietly substitute routing for boundary clarity
- fixed presentation replaces capability teaching

## Good Move

"Keep the law inline. Move the expanded examples, templates, review prompts, and deterministic helpers into the right support folders, then point to them clearly."

"Keep capability-first boundary judgment in the main file. Use support docs to deepen the current skill, not to replace boundary clarity with a hidden routing graph."

"Use action hints only when they improve judgment. Do not turn them into mandatory SOP or output-template law."

Why this works:
- the first-pass decision stays local
- support detail remains available
- the main file becomes shorter without becoming weaker
- support docs reinforce the same law instead of smuggling in skill-to-skill routing
- the package teaches flexible judgment instead of format compliance
