# Description Patterns

## Purpose

The `description` field exists for discovery. It answers: *Should I load this skill now?*

Treat the skill package as two layers:
- `description` — discovery copy: helps the agent decide whether to load the skill
- main `SKILL.md` — capability teaching: what the skill owns, what boundaries and invariants apply, what signals shape judgment

If the `description` carries boundary law, decision logic, or action guidance, it is doing the main file's job. If the main `SKILL.md` depends on the `description` to explain the capability, the package is upside down.

## Core Tests

Apply before writing or reviewing any description:
- Does it say *when* to use the skill, not *what steps* it performs?
- Does it help the agent *decide to load* the skill — not help them use it after loading?
- If you remove named skill references, does it remain meaningful? If not, it was depending on routing language instead of trigger conditions.

## Recommended Format

```yaml
description: "Invoke [when/unless] [observable condition]. Cost of unnecessary invocation: [specific minor cost]. Cost of missing: [specific major harm]."
```

**Quote the description when it contains colons.** YAML treats unquoted `: ` as a key-value separator, breaking frontmatter parsing. Always quote descriptions containing `Cost of unnecessary invocation:` or any `: ` sequence.

When costs are stated explicitly, the right default becomes obvious.

**Prefer "when" over "unless".** The "unless" form places an exclusion at the first gate, before the agent has investigated. Only use "unless" when the exclusion is a fully external, independently verifiable fact — not an assessment of the current task.

**"Do not use" conditions in the description face maximum rationalization pressure.** They are evaluated before the agent has invested in understanding the situation. Prefer moving them to the Trigger Logic body, where the agent reasons about them after loading the skill.

```yaml
# Best: positive "when" trigger with cost asymmetry
description: "Invoke when requirements are unclear or stakeholders have conflicting needs. Cost of unnecessary invocation: 2-3 clarifying questions. Cost of missing: building the wrong thing, requiring rework."

# Last resort: "unless" with a fully external, independently verifiable fact
description: "Invoke unless a completed plan document already exists at docs/plans/ for this task. Cost of unnecessary invocation: brief review. Cost of missing: starting without known files, scope, or dependencies."

# Bad: impression-based condition — "already obvious" is assessed before investigation
description: "Invoke unless the root cause is already obvious. Cost of..."
```

## Good Pattern

```yaml
description: "Invoke when creating or modifying a SKILL.md, or when a skill is triggering at the wrong time, teaching the wrong thing, or behaving differently than expected. Cost of unnecessary invocation: brief structural review. Cost of missing: a skill that silently misbehaves while appearing well-formed."
```

Why it works:
- describes observable states that trigger loading, not steps the skill performs
- cost asymmetry makes the default obvious
- no "Do not use" in the description — exclusions belong in the Trigger Logic body

## Bad Patterns

```yaml
# Workflow summary — describes steps, not trigger
description: This skill gathers notes, writes a first draft, compares outputs, revises the draft, and packages the final result.
```

Why it fails: describes process instead of trigger; can become a shortcut that replaces reading the main skill.

```yaml
# Mini skill — teaches capability instead of enabling discovery
description: Use when writing skills that should preserve clear boundaries, keep invariants inline, avoid routing language, and prefer contrast examples over rigid output templates.
```

Why it fails: compresses capability law into the wrong layer; increases the chance the agent never reads the actual skill.

## What Discovery Copy Must Not Do

Do not use the `description` to:
- summarize steps or workflow
- teach invariants or red lines
- explain how to perform the task
- encode a preferred output shape
- hide skill-to-skill routing

These belong in the main skill body if they belong anywhere.

## Rewrite Pattern

**Strip wrong content:**
1. Remove process summary
2. Remove capability law, invariants, and action guidance — these belong in the main file
3. Keep only trigger conditions

**Add what belongs:**
4. Add concrete contexts or symptoms the agent can recognize
5. Express the invocation default with explicit cost asymmetry
6. Ensure "Do not use" conditions (if any) are observable facts, not impressions

**Polish:**
7. Keep it concise enough to scan quickly
8. Confirm boundary language, invariants, and action guidance remain in the main file, not here

## Review Questions

- Does it say when to use the skill?
- Does it mention contexts or symptoms a user might actually express?
- Does it avoid summarizing the workflow?
- Does it avoid teaching the capability itself?
- Does it stay concise enough to be scanned quickly?

See `references/skill-trigger-design-principles.md` for the full trigger design framework.
