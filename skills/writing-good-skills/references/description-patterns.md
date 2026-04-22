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
- **Two different truncation boundaries exist — do not confuse them:**
  - **250 characters — UI display truncation.** When a user browses the skill list (e.g. `/skills`), each description is truncated at 250 chars in the listing. This is a display concern only.
  - **1024 characters — hard field limit.** The maximum length the `description` field accepts.
  - **The LLM reads the full description up to 1024 characters.** The 250-char truncation does NOT affect the model's ability to read the complete text. The model receives the full description in `<available_skills>`.
- The global budget across all skills is **~1% of context window** (minimum 8000 chars).
- The triggering decision is made entirely by LLM reasoning — no classifier, no embedding, no intent detection. The model reads descriptions as natural language and decides.
- This mechanism is shared across Claude Code, Codex CLI, and any system following the Agent Skills standard.

**Implication: front-load capability + TRIGGER conditions within the first 250 characters so they survive UI display truncation. DO NOT compress the entire description to fit within 250 characters** — doing so would sacrifice DO NOT TRIGGER disambiguation boundaries that the LLM reads and uses for routing. A description of 250–350 characters that front-loads correctly is better than a 240-character description that omits disambiguation.

## Core Tests

Apply before writing or reviewing any description. Each test has a verifiable pass/fail form.

**Test 1: When, not how**
Does the description state *when* to trigger, not *what steps* to perform?

- Pass: `"TRIGGER when: a diff or PR exists and the task is to review it"`
- Fail: `"This skill gathers notes, writes a first draft, compares outputs, and packages the result"` — describes steps, not trigger state

**Test 2: Routes to load, not how to use**
Does it help the agent decide *whether to load* the skill — not how to use it after loading?

- Pass: `"Evaluate code changes for correctness, design, and risk"`
- Fail: `"Use this skill's Boundary section to classify the change, then apply the relevant invariant"` — teaches usage, not routing

**Test 3: Trigger within 250 characters**
Does capability + TRIGGER condition appear before the 250-char display cutoff?

- Pass: capability statement (~60 chars) + `TRIGGER when` (~80 chars) = 140 chars total — key info survives truncation
- Fail: 200-char preamble about cost and context, TRIGGER condition buried after char 250 — invisible in UI listing

**Test 4: Meaningful after removing named-skill references**
Remove all "use X instead" phrases. Does the description still identify what state triggers this skill?

- Pass: description still identifies the trigger state independently — sibling references were additive, not load-bearing
- Fail: description becomes `"Use when not doing what receiving-code-review does"` — collapses after removal; the positive trigger was never stated

## Recommended Format

```yaml
description: "[One-line capability statement]. TRIGGER when: [observable conditions]. DO NOT TRIGGER when: [disambiguation boundaries]."
```

**Three components, in this order:**

1. **What** (capability) — one sentence, front-loaded. This is what the model reads first and what survives truncation.
2. **TRIGGER when** — specific, observable conditions. Use concrete terms the user would naturally say (file extensions, task types, domain keywords).
3. **DO NOT TRIGGER when** — sibling skill disambiguation only. Include when a named sibling skill has overlapping trigger conditions. Points the model to the correct alternative. If no sibling skill exists, fix the positive TRIGGER instead of adding this clause.

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

```yaml
# Compensatory DO NOT TRIGGER — masking an imprecise positive trigger
description: "Guide error handling decisions. TRIGGER when: you need to decide how to handle an error. DO NOT TRIGGER when: the correct approach is already clear."
```

Why it fails: the positive TRIGGER (`"you need to decide how to handle an error"`) is too broad and would match almost any error-adjacent task. The `DO NOT TRIGGER` clause attempts to compensate with an impression-based guard (`"already clear"`) — which is assessed before reading code. The fix is to narrow the positive TRIGGER (e.g., state the specific error categories, file types, or decision context it covers), not to add an exclusion.

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
9. Ensure capability + TRIGGER conditions are in the first 250 characters (UI display truncation)
10. Keep total under 1024 characters (hard limit)
11. Do NOT compress to fit within 250 characters — 250–350 chars with proper front-loading and DO NOT TRIGGER is better than 240 chars without disambiguation
11. Confirm boundary language, invariants, and action guidance remain in the main file, not here

## DO NOT TRIGGER: When to Include, What to Include

`DO NOT TRIGGER` has exactly one legitimate job: route to the correct skill when two skills genuinely have overlapping positive trigger conditions. It is not a tool for reducing invocation frequency, adding impression-based guards, or compensating for an imprecise positive trigger.

**Legitimacy test** — run before writing any DO NOT TRIGGER clause:

Remove the DO NOT TRIGGER clause. Can the agent route correctly from the positive TRIGGER alone?

- **Yes** → DO NOT TRIGGER is unnecessary. Delete it.
- **No, a named sibling skill has overlapping trigger conditions** → DO NOT TRIGGER is legitimate disambiguation. Keep it, name the sibling.
- **No, no sibling skill exists** → The positive TRIGGER is imprecise. Fix the TRIGGER; do not add DO NOT TRIGGER to compensate.

**Form matters:**

Pass: `DO NOT TRIGGER when: responding to existing review feedback (use receiving-code-review)` — observable state + named sibling

Fail: `DO NOT TRIGGER when: the change is simple enough to not need this` — impression assessed before investigation; fix the positive TRIGGER instead

Fail: `DO NOT TRIGGER when: you already know what to do` — no named sibling, no observable fact; this is a rationalization exit, not disambiguation

## Review Questions

- Does the capability statement appear in the first 60 characters?
- Does it say when to use the skill, using observable states?
- Does it disambiguate from similar skills where confusion is realistic?
- Does it avoid summarizing the workflow or teaching the capability?
- Is the most critical info within 250 characters?
- Is the total under 1024 characters?

See `references/skill-trigger-design-principles.md` for the full trigger design framework.
