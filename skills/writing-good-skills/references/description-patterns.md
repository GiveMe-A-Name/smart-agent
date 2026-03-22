# Description Patterns

## Purpose

The `description` field exists for discovery.

It should help the agent decide whether to load the skill. It should not summarize the full workflow.

## Good Pattern

Write in third person. Describe:
- when the skill applies
- what kind of task is happening
- what symptoms or contexts should trigger loading

Example:

```yaml
description: Use when creating or revising a reusable skill package, especially from rough notes, from-scratch skill ideas, vague trigger wording, boundary confusion with `AGENTS.md` or `README.md`, or missing realistic evals.
```

Why it works:
- it describes use conditions
- it names realistic contexts
- it does not tempt the agent to skip the body

## Bad Pattern

Do not summarize the workflow in the description.

Example:

```yaml
description: This skill gathers notes, writes a first draft, compares outputs, revises the draft, and packages the final result.
```

Why it fails:
- it describes process instead of trigger
- it can become a shortcut that replaces reading the main skill
- it weakens discoverability

## Review Questions

Check these when reviewing a description:
- does it say when to use the skill?
- does it mention contexts or symptoms a user might actually express?
- does it avoid summarizing the workflow?
- does it stay concise enough to be scanned quickly?

## Rewrite Pattern

When rewriting a weak description:
1. remove process summary
2. keep only trigger conditions
3. add concrete contexts or symptoms
4. keep it concise

## Minimal Heuristic

If the description answers "what steps does this skill perform?" more than "when should this skill load?", rewrite it.
