# Description Patterns

Use this reference when creating or optimizing a skill's frontmatter `description`.

The `description` is a discovery field, not a skill summary. It helps the agent decide whether to load the skill before the full `SKILL.md` is available. The main `SKILL.md` teaches the capability after loading.

## Description Shape

Use the `description` for pre-load discovery:

Default shape:

```yaml
description: "[Capability]. Use when [observable invocation state]."
```

The capability sentence answers what behavior the skill improves. The `Use when` sentence gives the observable request terms, artifact types, or task states that should make the agent load it.

Avoid negative trigger clauses by default. If the positive trigger would still load the skill for a concrete task it does not own, first narrow the `Use when` clause. Add a short scope boundary only when narrowing would hide a real positive trigger.

Do not put cost-asymmetry persuasion, workflow steps, or skill-body rules in the `description`. If the skill needs richer uncertainty handling, encode it in the main `SKILL.md` as a principle or constraint after the skill has loaded:

```markdown
- When [safe skip condition] is not confirmed from evidence, continue applying this skill [because the cost of a short extra reasoning pass is lower than the cost of missing the capability when it is needed].
```

## Description Rules

A good description contains two required parts, in this order:

- **Capability**: one short statement of what behavior the skill improves.
- **Use when**: observable conditions that make the skill worth loading.

Use terms the agent is likely to see in the user request or workspace: file extensions, artifact names, task verbs, domain objects, and task states.

Quote descriptions that contain `: ` so YAML does not parse them as nested keys.

## Calibrated Examples

```yaml
description: "Extract text and tables from PDF files, fill forms, and merge documents. Use when working with PDF files, forms, or document extraction."
```

```yaml
description: "Analyze spreadsheets, create pivot tables, and generate charts. Use when analyzing Excel files, .xlsx files, spreadsheets, or tabular data."
```

```yaml
description: "Generate commit messages from git diffs. Use when the user asks for a commit message or help reviewing staged changes."
```

These work because they name the capability first, then give concrete loading evidence without teaching the workflow.

## Constraints

- Put the capability and main `Use when` clause early; UI lists may truncate around 250 characters.
- The field limit is 1024 characters; the model can read the full description up to that limit.
- Do not compress the whole description to 250 characters if doing so removes necessary boundary information.
- The load decision is made from `name` and `description` before the full `SKILL.md` is loaded.

## Scope Boundary Principles

Most descriptions should have no boundary clause. Prefer a narrower positive `Use when` clause over a negative exclusion.

If a boundary is unavoidable, it must be based on observable state, not pre-investigation confidence.

Delete or rewrite conditions that depend on judgments such as:

- obvious
- simple
- trivial
- tiny
- already clear
- low risk

These words let the agent skip the skill using the same shallow judgment the skill is meant to protect against.

If a safe-skip condition can only be known after reading files, tracing behavior, or checking the artifact, do not put it in the frontmatter `description`; express it as a constraint in the main `SKILL.md`:

```markdown
- Stop applying this skill only after confirming the entire change is limited to one function and has no callers outside the file.
```

Do not hardcode named-skill handoffs in the `description`. If another skill should have loaded first, fix that skill's frontmatter `description` instead of teaching this skill to route to it.

## Good Default Pattern

```yaml
description: "Create and refine Codex skills that teach reusable agent judgment. Use when creating a new skill, optimizing an existing skill, or improving a skill description."
```

Why it works:

- starts with the capability
- uses observable state: creating or optimizing a skill
- omits a boundary because the positive `Use when` clause is already narrow enough
- does not explain how to write the skill

## Good Pattern With Narrow Positive Trigger

```yaml
description: "Evaluate code changes for correctness, design, and merge risk. Use when a diff, staged changes, or PR exists and the user asks to review, audit, or judge those changes."
```

Why it works:

- starts with the capability
- uses observable state: diff, staged changes, PR, review task
- narrows ownership with positive action words: review, audit, judge
- does not explain how to perform the review

## Boundary Rule

Add a scope boundary only when it prevents a concrete wrong invocation that the positive `Use when` clause cannot prevent.

Before adding one, all answers must be yes:

- Have you already narrowed the positive `Use when` clause as much as possible without hiding valid trigger cases?

- Is the boundary based on observable state rather than agent confidence or perceived simplicity?

- Would deleting the boundary make the skill load for a task it does not own?

- Can the boundary be phrased as scope or ownership instead of a negative trigger instruction?

If any answer is no, rewrite the positive `Use when` clause instead of adding a boundary.
