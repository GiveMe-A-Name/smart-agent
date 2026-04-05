# Description Patterns

## Purpose

The `description` field is a **trigger, not a summary**. It answers one question: *Should I load this skill now?*

Treat the skill package as two layers:
- `description` — discovery trigger: helps the agent decide whether to load the skill
- main `SKILL.md` — capability teaching: what the skill owns, what boundaries and invariants apply, what signals shape judgment

If the `description` carries boundary law, decision logic, or action guidance, it is doing the main file's job. If the main `SKILL.md` depends on the `description` to explain the capability, the package is upside down.

## Technical Context

Understanding how descriptions are consumed affects how to write them:

- All skill descriptions are loaded into the agent's context at session start — only `name` + `description`, not the full SKILL.md body. The full body loads only after the agent decides to invoke.
- Display truncation occurs at **250 characters**. The hard limit is **1024 characters**. The global budget is **~1% of context window**.
- The triggering decision is made entirely by LLM reasoning — no classifier, no embedding, no intent detection. The model reads descriptions as natural language and decides.
- This mechanism is shared across Claude Code, Codex CLI, and any system following the Agent Skills standard.

Implication: the first 250 characters must contain the most important trigger information. Everything after 250 chars may be truncated in skill listings.

## Core Tests

Apply before writing or reviewing any description:
- Does it say *when* to use the skill, not *what steps* it performs?
- Does it help the agent *decide to load* the skill — not help them use it after loading?
- Does the first 250 characters contain the primary trigger condition?
- If you remove named skill references, does it remain meaningful?

## Recommended Format

```yaml
description: "[One-line capability statement]. TRIGGER when: [observable conditions]. DO NOT TRIGGER when: [disambiguation boundaries]."
```

**Three components, in this order:**

1. **What** (capability) — one sentence, front-loaded. This is what the model reads first and what survives truncation.
2. **TRIGGER when** — specific, observable conditions. Use concrete terms the user would naturally say (file extensions, task types, domain keywords).
3. **DO NOT TRIGGER when** — disambiguation boundaries. Most valuable when similar skills exist. Points the model to the correct alternative.

**Quote the description when it contains colons.** YAML treats unquoted `: ` as a key-value separator, breaking frontmatter parsing. Always quote descriptions that use the TRIGGER/DO NOT TRIGGER format.

### Why This Format

The old `Cost of unnecessary invocation: ... Cost of missing: ...` pattern consumed 80-120 characters on persuasion that doesn't help the triggering decision. The model doesn't need to be convinced the skill is valuable — it needs to decide whether the current situation matches.

`DO NOT TRIGGER when` in the description serves a different purpose than `Do not use` conditions in the Trigger Logic body:
- **In description**: disambiguation between similar skills at routing time. The model is choosing which skill to load, and it needs to know the boundary between skill A and skill B.
- **In Trigger Logic body**: nuanced exclusions that require reasoning after the skill is loaded.

The earlier guidance ("Do not use conditions in the description face maximum rationalization pressure") remains valid for impression-based exclusions like "unless the fix is obvious." But for factual disambiguation — "DO NOT TRIGGER when responding to existing review feedback (use receiving-code-review)" — the description is exactly where this belongs, because the model makes the routing decision before loading either skill.

## Good Patterns

```yaml
# What + TRIGGER + DO NOT TRIGGER with disambiguation
description: "Evaluate code changes for correctness, design, and risk. TRIGGER when: a diff, staged changes, or PR exists and the task is to review them. DO NOT TRIGGER when: responding to existing review feedback (use receiving-code-review) or no code changes exist yet."
```

Why it works:
- capability statement in first 60 chars
- TRIGGER uses observable state ("diff exists")
- DO NOT TRIGGER disambiguates from a similar skill by name

```yaml
# Concise, no disambiguation needed
description: "Find root causes, not just fixes — systematic diagnosis before changing code. TRIGGER when: something is broken or behaving unexpectedly and the fix is not yet confirmed. DO NOT TRIGGER when: building a new feature or refactoring with no current failure."
```

Why it works:
- core value proposition front-loaded ("root causes, not just fixes")
- trigger is state-based, not scenario-based
- exclusion is an observable fact ("no current failure"), not an impression

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

```yaml
# Cost template consuming space without aiding trigger decision
description: "Invoke when X. Cost of unnecessary invocation: a short review pass. Cost of missing: bugs in production or false safety from an incomplete review."
```

Why it fails: the cost sentences consume ~120 characters that could carry disambiguation boundaries or concrete trigger keywords. The model doesn't need persuasion — it needs routing precision.

```yaml
# Impression-based exclusion — assessed before investigation
description: "Invoke unless the root cause is already obvious."
```

Why it fails: "already obvious" is judged before reading code. This is a rationalization exit, not a disambiguation boundary.

## What Discovery Copy Must Not Do

Do not use the `description` to:
- summarize steps or workflow
- teach invariants or red lines
- explain how to perform the task
- encode a preferred output shape
- hide skill-to-skill routing (naming another skill as "invoke X next" is routing; naming it as "use X instead" in DO NOT TRIGGER is disambiguation)

These belong in the main skill body if they belong anywhere.

## Rewrite Pattern

**Strip wrong content:**
1. Remove process summary
2. Remove capability law, invariants, and action guidance — these belong in the main file
3. Remove cost template sentences
4. Keep only trigger conditions and capability statement

**Add what belongs:**
5. Front-load a one-line capability statement (what the skill does)
6. Add TRIGGER when with observable, state-based conditions
7. Add DO NOT TRIGGER when with disambiguation boundaries (especially if similar skills exist)
8. Use concrete keywords the user would naturally say (file types, task names, domain terms)

**Polish:**
9. Ensure the most critical trigger info is in the first 250 characters
10. Keep total under 1024 characters (hard limit)
11. Confirm boundary language, invariants, and action guidance remain in the main file, not here

## DO NOT TRIGGER: When to Include, What to Include

**Include when:**
- A sibling skill covers a closely related trigger state (code-review vs. receiving-code-review)
- The positive TRIGGER condition is broad enough that a realistic wrong-invocation exists
- The boundary is an observable fact ("no code changes exist yet"), not an impression ("the change is trivial")

**Omit when:**
- The positive TRIGGER condition is precise enough that no realistic confusion exists
- The exclusion would be impression-based ("unless the problem is obvious")
- The only purpose is to reduce invocation frequency, not to route to the correct skill

**Form matters:**
- Good: `DO NOT TRIGGER when: responding to existing review feedback (use receiving-code-review)` — observable fact + alternative
- Bad: `DO NOT TRIGGER when: the change is simple enough to not need this` — impression, rationalization exit

## Review Questions

- Does the capability statement appear in the first 60 characters?
- Does it say when to use the skill, using observable states?
- Does it disambiguate from similar skills where confusion is realistic?
- Does it avoid summarizing the workflow or teaching the capability?
- Is the most critical info within 250 characters?
- Is the total under 1024 characters?

See `references/skill-trigger-design-principles.md` for the full trigger design framework.
