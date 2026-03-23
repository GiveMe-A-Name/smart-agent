# Description Patterns

## Purpose

The `description` field exists for discovery.

It should help the agent decide whether to load the skill. It should not summarize the full workflow or hide capability law behind process shorthand.

## Good Pattern

Write in third person. Describe:
- when the skill applies
- what kind of task is happening
- what symptoms or contexts should trigger loading

Keep the description capability-first. It should help the agent recognize the skill's boundary and trigger, not pre-read the workflow.

Example:

```yaml
description: Use when creating or revising a reusable skill package, especially from rough notes, overlong or repetitive `SKILL.md` drafts, weak trigger wording, unclear main-file versus support-folder splits (`references/`, `templates/`, `examples/`, `scripts/`), or boundary confusion with `AGENTS.md` or `README.md`.
```

Why it works:
- it describes use conditions
- it names realistic contexts
- it does not tempt the agent to skip the body
- it supports capability-first discovery instead of process-first shorthand

## Bad Pattern

Do not summarize the workflow in the description.

Do not use the description to smuggle in boundary law or skill-to-skill routing.

Example:

```yaml
description: This skill gathers notes, writes a first draft, compares outputs, revises the draft, and packages the final result.
```

Why it fails:
- it describes process instead of trigger
- it can become a shortcut that replaces reading the main skill
- it weakens discoverability
- it teaches a workflow outline instead of capability-first judgment

## Review Questions

Check these when reviewing a description:
- does it say when to use the skill?
- does it mention contexts or symptoms a user might actually express?
- does it avoid summarizing the workflow?
- does it stay concise enough to be scanned quickly?
- does it preserve capability-first discovery instead of trying to teach the whole skill?

## Rewrite Pattern

When rewriting a weak description:
1. remove process summary
2. keep only trigger conditions
3. add concrete contexts or symptoms
4. keep it concise
5. keep boundary language and core law in the main file, not in the description

## Minimal Heuristic

If the description answers "what steps does this skill perform?" more than "when should this skill load?", rewrite it.

If removing named skill references makes the description vague, it was probably depending on routing language instead of clear trigger conditions.
