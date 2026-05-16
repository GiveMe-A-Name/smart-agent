# Description Patterns

Use this reference when creating or optimizing a skill's frontmatter `description` or trigger logic.

The `description` is a discovery trigger, not a skill summary. It helps the agent decide whether to load the skill before the full `SKILL.md` is available. The main `SKILL.md` teaches the capability after loading.

## Description vs Trigger Logic Body

Use the `description` for pre-load routing:

```yaml
description: "[Capability]. TRIGGER when: [observable invocation state]. DO NOT TRIGGER when: [observable boundary, only if needed]."
```

Use the `Trigger Logic` body for richer invocation defaults and skip rules after the skill has loaded.

Do not put cost-asymmetry persuasion in the description. If the skill needs an uncertainty default, put it in the body:

```markdown
**Invocation default**: The cost of unnecessary invocation is [minor overhead]. The cost of missed invocation is [specific downstream harm]. When [safe skip condition] is not confirmed from evidence, invoke.
```

## Description Rules

A good description contains three parts, in this order:

- **Capability**: one short statement of what behavior the skill improves.
- **TRIGGER when**: observable conditions that make the skill worth loading.
- **DO NOT TRIGGER when**: only use this for concrete boundary cases that the positive trigger cannot make clear by itself.

Quote descriptions that contain `: ` so YAML does not parse them as nested keys.

## Constraints

- Put the capability and main trigger early; UI lists may truncate around 250 characters.
- The field limit is 1024 characters; the model can read the full description up to that limit.
- Do not compress the whole description to 250 characters if doing so removes necessary boundary information.
- The trigger decision is made from `name` and `description` before the full `SKILL.md` is loaded.

## Trigger Logic Principles

`Do not use` and `DO NOT TRIGGER` conditions must be based on observable state, not pre-investigation confidence.

Delete or rewrite conditions that depend on judgments such as:

- obvious
- simple
- trivial
- tiny
- already clear
- low risk

These words let the agent skip the skill using the same shallow judgment the skill is meant to protect against.

If the safe-skip condition can only be known after reading files, tracing behavior, or checking the artifact, write it as a confirmed evidence condition:

```markdown
Do not use this skill only after confirming the entire change is limited to one function and has no callers outside the file.
```

Do not hardcode named-skill handoffs in trigger logic. If another skill should have triggered first, fix that skill's trigger or boundary instead of teaching this skill to route to it.

## Good Pattern

```yaml
description: "Evaluate code changes for correctness, design, and risk. TRIGGER when: a diff, staged changes, or PR exists and the task is to review them. DO NOT TRIGGER when: responding to existing review feedback."
```

Why it works:

- starts with the capability
- uses observable state: diff, staged changes, PR, review task
- boundary is factual, not impression-based
- does not explain how to perform the review

## Bad Patterns

Workflow summary:

```yaml
description: "Gather notes, draft the skill, compare alternatives, revise examples, and package the result."
```

Why it fails: it describes steps instead of the state that should load the skill.

Mini skill body:

```yaml
description: "Use this skill to preserve clear boundaries, keep invariants inline, avoid routing language, and prefer contrast examples."
```

Why it fails: it teaches rules that belong in the main `SKILL.md`.

Impression-based exclusion:

```yaml
description: "TRIGGER when debugging. DO NOT TRIGGER when the fix is obvious."
```

Why it fails: "obvious" is self-judgment before investigation. Narrow the positive trigger instead.

Compensating boundary:

```yaml
description: "TRIGGER when working on documentation. DO NOT TRIGGER when this is not really a documentation task."
```

Why it fails: the positive trigger is too broad, and the exclusion does not name an observable boundary.

Hidden routing:

```markdown
Do not use this skill until you have invoked `some-other-skill` first.
```

Why it fails: explicit handoffs hide that `some-other-skill` needs a better trigger.

## DO NOT TRIGGER Rule

Add `DO NOT TRIGGER` only when it prevents a concrete wrong invocation that the positive trigger cannot prevent.

Before adding it, ask:

Can the positive trigger be narrowed instead?

Is the exclusion based on observable state rather than agent confidence or perceived simplicity?

Would deleting the exclusion make the skill trigger for a task it does not own?

If the answer to any question is no, rewrite the positive trigger instead of adding `DO NOT TRIGGER`.

## Final Check

Before accepting a description or trigger change, ask:

Does it say when to load the skill, not how to use it?

Can the trigger be recognized from the user's request or visible task state?

Does it avoid capability law, workflow steps, output templates, and hidden skill routing?

Are skip conditions based on confirmed evidence rather than confidence words like "obvious" or "simple"?

Is uncertainty handled in the body rather than by weakening the description?

Is the most important trigger information early enough to survive UI truncation?

Does the full description stay under 1024 characters?
