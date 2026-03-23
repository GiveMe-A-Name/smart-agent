# Output Shapes

## When Creating A Skill

Default output shape:
- capability-first boundary decision
- trigger logic explanation
- lean `SKILL.md`
- supporting folders only if justified
- short explanation of what behavior the skill should change

If the skill is complex, also include:
- a split map explaining what stays inline versus what moves to `references/`, `templates/`, `examples/`, or `scripts/`
- a short note on what repetition was removed or compressed

## When Reviewing A Skill

Default output shape:
- overall assessment
- what already works
- highest-impact gaps
- section review for description, scope, constitution, execution, packaging, and redundancy
- boundary language versus skill-to-skill routing issues if present
- prioritized fixes
- rewritten text only for the weakest sections

## When Rewriting Only Part Of A Skill

Do not reflexively rewrite the whole file.

Prefer:
- identify the weakest section
- explain why it is weak
- provide replacement text for that section
- state what effect the rewrite should have

If the weakness is about scope, prefer boundary language over handoff language or other skill-to-skill routing wording.

## Packaging Rule

Use support folders only when they improve decision quality in the main file.

Use `references/lean-skill-patterns.md` as the canonical split guide.

For this output file, do not re-list the full folder taxonomy or the full keep-inline law unless the current rewrite depends on a specific distinction.

Output-specific rule:
- when a split map is needed, point to the canonical guide and explain only the skill-specific exceptions, tradeoffs, or file-role decisions for the current package
- when a routing smell is present, name it as skill-to-skill routing and rewrite it in capability-first boundary terms
