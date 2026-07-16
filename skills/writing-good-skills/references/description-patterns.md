# Description Patterns

Use this reference whenever creating or changing a skill's frontmatter `description`.

The description is a discovery interface. Before reading `SKILL.md`, the agent uses it to decide whether the capability could improve the current task. Describe user intent and observable task state, not the skill's internal workflow.

## Default Shape

```yaml
description: "[Capability]. Use when [observable requests, artifacts, or task states]."
```

A useful description contains:

- **Capability**: the behavior the skill improves.
- **Trigger evidence**: terms, artifacts, requests, or task states visible before loading the body.
- **Boundary**, only when a realistic near miss remains after narrowing the positive trigger.

Front-load the capability and strongest triggers. Use concrete nouns and verbs likely to appear in real requests. Keep workflow steps, rationale, and post-load decisions in the body.

## Examples

```yaml
description: "Analyze spreadsheets, modify formulas and formatting, and create charts. Use when working with .xlsx, .xls, .csv, or tabular spreadsheet data."
```

```yaml
description: "Evaluate changed code for correctness, design, and merge risk. Use when a diff, staged changes, or pull request exists and the task is to review, audit, or judge those changes."
```

The second example does not trigger on every mention of code or pull requests; it owns evaluation of an existing change.

## Boundary Rules

Prefer a precise positive trigger over a list of exclusions. Add a boundary only when all three are true:

1. A realistic adjacent task shares the description's keywords.
2. Loading this skill for that task would add the wrong method or cost.
3. The distinction is observable before loading `SKILL.md`.

Do not use perceived simplicity, confidence, or risk as pre-load conditions when they require investigation. Do not route to named skills from the description; each skill should describe its own ownership.

## Trigger Evaluation

Create realistic prompts covering:

- direct requests and indirect phrasings that should trigger;
- terse and context-heavy prompts;
- the capability embedded in a larger task;
- near misses with overlapping artifacts or terminology;
- adjacent tasks owned by a different capability.

Run prompts multiple times when selection is nondeterministic. Revise general categories, not individual test wording, and keep unseen validation prompts to detect overfitting.

