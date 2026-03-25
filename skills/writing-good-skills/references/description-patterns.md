# Description Patterns

## Purpose

The `description` field exists for discovery.

It should help the agent decide whether to load the skill. It should not summarize the full workflow, teach the capability itself, or hide capability law behind process shorthand.

## Two Layers, Two Jobs

Treat the skill package as two different layers:

- `description` = discovery copy
- main `SKILL.md` = capability teaching

The `description` answers: `Should I load this skill now?`

The main `SKILL.md` answers:
- what capability this skill owns
- what boundaries and invariants matter
- what signals should shape judgment
- what failure modes to avoid

If the `description` starts carrying boundary law, decision logic, or action guidance, it is doing the main file's job.

If the main `SKILL.md` depends on the `description` to explain the capability, the package is upside down.

## Good Pattern

Write in third person. Describe:
- when the skill applies
- what kind of task is happening
- what symptoms or contexts should trigger loading

Keep the description capability-first. It should help the agent recognize the skill's boundary and trigger, not pre-read the workflow.

Think of it as label text on the drawer, not the tool inside the drawer.

Example:

```yaml
description: Use when creating or substantially revising a reusable skill package whose trigger wording, boundary, invariants, examples, or support-file split need structural judgment. Do not use for isolated wording tweaks, obvious local edits, or plain project documentation.
```

Why it works:
- it describes use conditions
- it names realistic contexts
- it does not tempt the agent to skip the body
- it supports capability-first discovery instead of process-first shorthand

## What Discovery Copy Must Not Do

Do not use the `description` to:
- summarize steps
- teach invariants or red lines
- explain how to perform the task
- encode a preferred output shape
- hide skill-to-skill routing

Those belong in the main skill body if they belong anywhere at all.

## Bad Pattern

Do not summarize the workflow in the description.

Do not use the description to smuggle in boundary law or skill-to-skill routing.

Do not use the description as a mini skill.

Example:

```yaml
description: This skill gathers notes, writes a first draft, compares outputs, revises the draft, and packages the final result.
```

Why it fails:
- it describes process instead of trigger
- it can become a shortcut that replaces reading the main skill
- it weakens discoverability
- it teaches a workflow outline instead of capability-first judgment

Another failure mode:

```yaml
description: Use when writing skills that should preserve clear boundaries, keep invariants inline, avoid routing language, and prefer contrast examples over rigid output templates.
```

Why it fails:
- it sounds sophisticated, but it is teaching the skill instead of helping discovery
- it compresses boundary law and writing philosophy into the wrong layer
- it increases the chance that the agent never reads the actual skill

## Review Questions

Check these when reviewing a description:
- does it say when to use the skill?
- does it mention contexts or symptoms a user might actually express?
- does it avoid summarizing the workflow?
- does it avoid teaching the capability itself?
- does it stay concise enough to be scanned quickly?
- does it preserve capability-first discovery instead of trying to teach the whole skill?

## Rewrite Pattern

When rewriting a weak description:
1. remove process summary
2. remove capability law that belongs in the main file
3. keep only trigger conditions
4. add concrete contexts or symptoms
5. keep it concise
6. keep boundary language, invariants, and action guidance in the main file, not in the description

## Fast Separation Test

Ask two questions:

1. does this phrase help an agent decide to load the skill?
2. or does it help the agent use the skill after loading it?

If it helps with loading, it may belong in the `description`.

If it helps with using, judging, or applying the skill, it belongs in the main `SKILL.md`.

## Minimal Heuristic

If the description answers "what steps does this skill perform?" more than "when should this skill load?", rewrite it.

If the description answers "what should I believe or do once the skill is loaded?", rewrite it.

If removing named skill references makes the description vague, it was probably depending on routing language instead of clear trigger conditions.
