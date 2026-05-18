# Description Patterns

Use this reference when creating or optimizing a skill's frontmatter `description`.

The `description` is a discovery field, not a skill summary. It helps the agent decide whether to load the skill before the full `SKILL.md` is available. The main `SKILL.md` teaches the capability after loading.

## Description Shape

Use the `description` for pre-load discovery:

Without a boundary:

```yaml
description: "[Capability]. TRIGGER when: [observable invocation state]."
```

With an optional boundary:

```yaml
description: "[Capability]. TRIGGER when: [observable invocation state]. DO NOT TRIGGER when: [optional observable boundary]."
```

Do not put cost-asymmetry persuasion, workflow steps, or skill-body rules in the `description`. If the skill needs richer uncertainty handling, encode it in the main `SKILL.md` as a principle or constraint after the skill has loaded:

```markdown
- When [safe skip condition] is not confirmed from evidence, continue applying this skill [because the cost of a short extra reasoning pass is lower than the cost of missing the capability when it is needed].
```

## Description Rules

A good description contains two required parts and one optional boundary part, in this order:

- **Capability**: one short statement of what behavior the skill improves.
- **TRIGGER when**: observable conditions that make the skill worth loading.
- **DO NOT TRIGGER when** (optional): add only for concrete boundary cases that the positive trigger cannot make clear by itself.

Quote descriptions that contain `: ` so YAML does not parse them as nested keys.

## Constraints

- Put the capability and main `TRIGGER when` clause early; UI lists may truncate around 250 characters.
- The field limit is 1024 characters; the model can read the full description up to that limit.
- Do not compress the whole description to 250 characters if doing so removes necessary boundary information.
- The load decision is made from `name` and `description` before the full `SKILL.md` is loaded.

## Boundary Clause Principles

`DO NOT TRIGGER` conditions must be based on observable state, not pre-investigation confidence.

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

## Good Pattern Without Boundary

```yaml
description: "Create and refine Codex skills that teach reusable agent judgment. TRIGGER when creating a new skill or optimizing an existing skill."
```

Why it works:

- starts with the capability
- uses observable state: creating or optimizing a skill
- omits `DO NOT TRIGGER` because the positive `TRIGGER when` clause is already narrow enough
- does not explain how to write the skill

## Good Pattern With Boundary

```yaml
description: "Evaluate code changes for correctness, design, and risk. TRIGGER when: a diff, staged changes, or PR exists and the task is to review them. DO NOT TRIGGER when: responding to existing review feedback."
```

Why it works:

- starts with the capability
- uses observable state: diff, staged changes, PR, review task
- boundary is factual, not impression-based
- does not explain how to perform the review

## DO NOT TRIGGER Rule

Add `DO NOT TRIGGER` only when it prevents a concrete wrong invocation that the positive `TRIGGER when` clause cannot prevent.

Before adding it, ask:

Can the positive `TRIGGER when` clause be narrowed instead?

Is the exclusion based on observable state rather than agent confidence or perceived simplicity?

Would deleting the exclusion make the skill load for a task it does not own?

If the answer to any question is no, rewrite the positive `TRIGGER when` clause instead of adding `DO NOT TRIGGER`.
