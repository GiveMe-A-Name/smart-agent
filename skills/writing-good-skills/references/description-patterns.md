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
description: "Invoke when creating or modifying a SKILL.md, or when a skill is triggering at the wrong time, teaching the wrong thing, or behaving differently than expected. Cost of unnecessary invocation: brief structural review. Cost of missing: a skill that silently misbehaves while appearing well-formed."
```

Why it works:
- it describes the observable states that trigger loading — not what steps the skill performs
- it names concrete contexts an agent can recognize before loading the skill
- cost asymmetry makes the default obvious: the risk of skipping is high, the cost of loading is low
- no "Do not use" in the description — exclusions (e.g., "not for wording-only tweaks") belong in the Trigger Logic body, where the agent has already committed to reading the skill

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

## Invocation Default Pattern

A strong description expresses the **invocation default** — the direction in which uncertainty resolves.

Most skills should lean toward invoking. State the cost asymmetry directly to make the default obvious:

```yaml
# Strong: positive "when" trigger with cost asymmetry
description: "Invoke before writing any code. Cost of unnecessary invocation: brief situation assessment. Cost of missing: starting without knowing files, scope, or dependencies — discovered mid-implementation when work must be redone."
```

```yaml
# Weak: burden shifted to AI to judge what "substantial" means
description: Use when the work is substantial enough to need a plan.
```

The first form makes the default obvious from the costs. The second leaves "substantial enough" for the AI to judge — exactly the judgment the skill is meant to provide.

### Recommended Format Template

Use this format to make cost asymmetry explicit:

```yaml
description: "Invoke [when/unless] [observable condition]. Cost of unnecessary invocation: [specific minor cost]. Cost of missing: [specific major harm]."
```

**Quote the description when it contains colons.** YAML treats an unquoted `: ` as a key-value separator, which breaks frontmatter parsing in tools like `npx skills`. Wrap the entire description in double quotes whenever it contains `Cost of unnecessary invocation:` or any other `: ` sequence.

**Important: Prefer "when" over "unless" in descriptions.** The "unless" form introduces a "Do not use" condition at the first gate, where rationalization pressure is highest. Only use "unless" when the exclusion condition is a fully external, verifiable fact — not an assessment made about the current task.

Examples:

```yaml
# Best: positive "when" trigger with cost asymmetry — no exclusion in description
description: "Invoke when requirements are unclear or stakeholders have conflicting needs. Cost of unnecessary invocation: 2-3 clarifying questions. Cost of missing: building the wrong thing, requiring rework."
```

```yaml
# Last resort — "unless" with a fully external, independently verifiable fact
# (A file either exists at the path or it does not. No task-specific judgment required.)
description: "Invoke unless a completed plan document already exists at docs/plans/ for this task. Cost of unnecessary invocation: brief review. Cost of missing: starting without known files, scope, or dependencies."
```

```yaml
# Bad: impression-based condition — requires assessing the current task before loading
description: "Invoke unless the root cause is already obvious. Cost of..."
# Problem: "already obvious" is formed before investigation — exactly when the skill is most needed
```

The middle form ("unless" with external fact) is technically acceptable but rare. In practice: when in doubt, use "when" and move the exclusion to the Trigger Logic body. In the body, the agent has already committed to reading the skill — the condition becomes part of reasoning rather than a bypass gate.

This format forces explicit reasoning about:
- Observable trigger conditions (not impressions)
- The actual cost of false positives (usually minor)
- The actual cost of false negatives (usually major)

When the costs are made explicit, the right default becomes obvious.

**When in doubt, use "when" and move exclusion conditions to the Trigger Logic body** where the agent has already committed to reading the skill.

**"Do not use" conditions in the description are escape hatches — even observable ones.** The description is the first gate. It is evaluated before the agent has invested in understanding the situation, so any exclusion condition placed there faces maximum rationalization pressure with minimum context.

Prefer moving "Do not use" conditions to the Trigger Logic body. In the body, the agent has already committed to reading the skill; the condition becomes part of how the skill reasons rather than a bypass gate.

If a "Do not use" condition must appear in the description, it should be trivially confirmable from facts that exist independently of the current task — not assessments of the current situation. "A completed plan document exists at a specific path" is independent. "The root cause is already confirmed" is an impression the agent forms before investigating — exactly the kind of shortcut that causes trigger failures.

- Escape hatch: `"Do not use only when the root cause is already confirmed with evidence"` — "already confirmed" is an impression made before investigation; the agent will rationalize it as true and skip
- Better: move the condition to the Trigger Logic body, where the agent reasons about it after loading the skill, not before

See `references/skill-trigger-design-principles.md` for the full framework.

## Rewrite Pattern

When rewriting a weak description:
1. remove process summary
2. remove capability law that belongs in the main file
3. keep only trigger conditions
4. add concrete contexts or symptoms
5. express the invocation default explicitly
6. ensure "Do not use" conditions are observable facts, not impression judgments
7. keep it concise
8. keep boundary language, invariants, and action guidance in the main file, not in the description

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
