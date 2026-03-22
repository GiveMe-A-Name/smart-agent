# Output Shapes

## When Creating A Skill

Default output shape:
- skill boundary decision
- trigger logic explanation
- lean `SKILL.md`
- supporting `references/` only if justified
- realistic eval prompts
- short explanation of what behavior the skill should change

## When Reviewing A Skill

Default output shape:
- overall assessment
- what already works
- highest-impact gaps
- section review for description, scope, constitution, execution, verification, packaging
- prioritized fixes
- rewritten text only for the weakest sections

## When Rewriting Only Part Of A Skill

Do not reflexively rewrite the whole file.

Prefer:
- identify the weakest section
- explain why it is weak
- provide replacement text for that section
- state what effect the rewrite should have

## Packaging Rule

Use `references/` only when it improves decision quality in the main file.

Move out:
- expanded examples
- templates
- repeated review prompts
- long explanations

Keep inline:
- trigger logic
- boundary decisions
- constitution
- main workflow
- self-correction signals
