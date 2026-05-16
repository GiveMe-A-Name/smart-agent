# Progressive Disclosure Examples

Use these examples when editing a skill and you are deciding what must stay in the main `SKILL.md` versus what should move into `examples/`, `references/`, `scripts/`, or `assets/`.

Do not read this file for every skill edit. Read it when one of these conditions is true:
- the main `SKILL.md` is accumulating long examples, templates, or background explanation
- moving material out of the main file would leave only vague pointers or section headings
- a support file exists, but the main file does not say when an agent should read it
- the skill's critical trigger, boundary, invariant, completion, or failure-mode logic is no longer visible in the first read

## Example 1: Long examples belong outside the main file

Problem:

The main `SKILL.md` includes the core invariant and then embeds a full catalog of rewrites:

```markdown
- Trigger conditions must include inline rationale.

Bad trigger: "Use when working on code."
Good trigger: "Use when changing runtime behavior..."

Bad trigger: "Use when writing docs."
Good trigger: "Use when creating persistent technical artifacts..."

Bad trigger: "Use when debugging."
Good trigger: "Use when there is an observed failure..."

[seven more examples...]
```

Why this is wrong:

The agent must see the rule during invocation and execution, but it does not need the full catalog unless it is actively rewriting trigger conditions. Keeping every example inline makes the main file harder to scan and pushes later critical rules farther down.

Better main-file shape:

```markdown
- Trigger conditions must include inline rationale stating what problem occurs if the trigger is missed [because agents need the failure mechanism to apply the trigger in new scenarios].
```

Better support-file shape:

```markdown
# Trigger Rationale Examples

Read this file when rewriting trigger conditions or when a trigger reads like a topic label instead of an invocation rule.

## Broad trigger

Bad:
"Use when working on code."

Good:
"Use when changing runtime behavior and the correct implementation layer is not already proven..."
```

Observable test:

If deleting the support file would remove helpful examples but not remove the rule itself, the split is healthy.

## Example 2: Moving too much out makes the main file hollow

Problem:

The main `SKILL.md` replaces core law with pointers:

```markdown
## Trigger Logic

See `references/triggers.md`.

## Invariants

See `references/invariants.md`.

## Completion Criteria

See `references/checklist.md`.
```

Why this is wrong:

The first file no longer contains enough information for an agent to decide whether to invoke the skill or how to execute it safely. This violates progressive disclosure because the first layer is not a usable summary; it is only a table of contents.

Better main-file shape:

```markdown
## Trigger Logic

Use this skill when creating a new skill or optimizing an existing skill.

Do not use this skill for generic documentation-placement decisions unless the user has explicitly framed the work as skill creation or skill optimization.

## Invariants

- Skills teach judgment; they do not encode orchestration or hide routing graphs [because explicit skill handoffs can mask that the routed skill's trigger conditions are too broad, too narrow, or unclear].
- Every invariant, decision signal, completion criterion, and self-correction signal must be verifiable without additional judgment.

## Completion Criteria

- [ ] The trigger is state-based, includes inline rationale, and names the failure that occurs when it is missed.
- [ ] The boundary states what behavior the skill improves and what behavior it does not improve.
```

Better support-file use:

```markdown
See `examples/progressive-disclosure-examples.md` when deciding whether material belongs inline or in support files.
```

Observable test:

If an agent must open a support file before it can answer "Should I use this skill?", the main file is hollow.

## Example 3: Support files need explicit read conditions

Problem:

The main `SKILL.md` ends with a vague pointer:

```markdown
See `examples/` for more.
```

Why this is wrong:

The agent has no evidence-based condition for when to spend context on examples. It may either ignore relevant examples or read them by default, defeating progressive disclosure.

Better pointer:

```markdown
See `examples/progressive-disclosure-examples.md` when deciding what stays in the main `SKILL.md` versus what moves to examples or references.
```

Better support-file opening:

```markdown
Use these examples when editing a skill and you are deciding what must stay in the main `SKILL.md` versus what should move into support folders.
```

Observable test:

Every support-file pointer should answer: "What task state makes this file worth reading now?"

## Example 4: Templates and scripts should not be pasted into the main file

Problem:

The main `SKILL.md` contains a full reusable prompt, long checklist, shell script, or generated template.

```markdown
## Skill Template

---
name:
description:
---

# Title

## Trigger Logic

...

[full template continues for many lines]
```

Why this is wrong:

Templates and scripts are execution aids, not decision-critical law. Keeping them inline makes the skill slower to read and makes important behavioral constraints easier to miss.

Better split:

```markdown
Use `templates/skill-template.md` when creating a new skill from scratch and the user has not provided an existing file to edit.
```

Observable test:

If the material is copied, filled in, executed, or reused as a scaffold, it belongs in `templates/`, `scripts/`, or `assets/`, with only the read/use condition in the main file.

## Example 5: Expanded rationale moves out only after the rule is self-contained

Problem:

The main file contains a rule without enough reason to apply it in new cases:

```markdown
- Do not use named-skill handoffs.
```

The rationale is moved entirely to a reference file:

```markdown
See `references/routing.md` for why.
```

Why this is wrong:

The main rule is no longer self-contained. An agent can obey it mechanically in familiar cases but rationalize around it in new cases because the causal model is not visible.

Better main-file shape:

```markdown
- Skills teach judgment; they do not encode orchestration or hide routing graphs [because explicit skill handoffs can mask that the routed skill's trigger conditions are too broad, too narrow, or unclear; fix the routed skill's trigger instead of teaching another skill to route to it].
```

Better support-file role:

```markdown
See `examples/skill-routing-examples.md` when rewriting a draft that currently names another skill as the next action.
```

Observable test:

The main rule should still be actionable if the support file is never opened. The support file may add cases, contrasts, and rewrites, but it must not contain the only reason the rule exists.
