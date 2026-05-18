# Progressive Disclosure Examples

Use these examples when editing a skill and deciding what must stay in the main `SKILL.md` versus what should move into `examples/`, `references/`, `templates/`, `scripts/`, or `assets/`.

Do not read this file for every skill edit. Read it only when the inline-vs-support boundary is unclear, the main file is becoming long or repetitive, or a support-file pointer lacks a read condition.

Progressive disclosure means each layer remains useful on its own:
- frontmatter `description` decides whether the skill loads
- main `SKILL.md` guides execution through `## Principles` and `## Constraints`
- support files deepen specific cases only after the agent has a reason to see them

## Example 1: Give Every Support File A Read Condition

Bad pointer:

```markdown
See `examples/` for more.
See `references/` for details.
```

Why this fails:

The pointer does not say what task state makes the extra file worth reading. The agent will either chase references by default and waste context, or skip them when the example is needed.

Better pointer:

```markdown
See `examples/progressive-disclosure-examples.md` when deciding what stays in the main `SKILL.md` versus what moves to examples or references.
```

Better support-file see:

```markdown
Use these examples when editing a skill and deciding what must stay in the main `SKILL.md` versus what should move into `examples/`, `references/`, `templates/`, `scripts/`, or `assets/`.
```

Observable test:

Every support-file pointer should answer: "What observable edit state makes this file useful now?"

## Example 2: Choose The Support Folder By Job

Bad split:

```text
references/
  skill-template.md
  good-description.md
  rewrite-description.py
  progressive-disclosure-notes.md
```

Why this fails:

Everything was moved out of the main file, but the support layer is not organized by how the agent uses it. The agent has to inspect filenames to discover whether a file should be read, copied, compared, or executed.

Better split:

```text
references/
  description-patterns.md       # read for rules and edge cases
examples/
  progressive-disclosure-examples.md  # compare bad/better structures
templates/
  skill-template.md             # copy or adapt as a scaffold
scripts/
  validate_skill.py             # execute for deterministic checks
```

Observable test:

If the agent must see a support file to learn what kind of object it is, the folder choice is not doing enough work.

## Example 3: Move Expanded Rationale Only After The Rule Is Self-Contained

Bad split:

```markdown
## Principles

- Skills teach judgment; they do not encode orchestration.

See the routing reference for why.
```

Why this fails:

The visible principle is too weak to generalize. The support file contains the causal model, so an agent can obey the rule in familiar cases but rationalize around it in new cases.

Better main-file shape:

```markdown
- Skills teach judgment; they do not encode orchestration or hide routing graphs [because explicit skill handoffs can mask that the routed skill's frontmatter `description` is too broad, too narrow, or unclear; fix that description instead of teaching another skill to route to it].
```

Better support-file role:

```markdown
Use this reference when a draft already names other skills as next actions and you need contrast examples for rewriting those references into local principles or constraints.
```

Observable test:

The main rule should still be actionable if the support file is never opened. The support file may add cases and rewrites, but it must not contain the only reason the rule exists.
