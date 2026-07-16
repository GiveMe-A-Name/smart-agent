---
name: writing-good-skills
description: "Create and refine capability skills that turn unstable agent behavior into reusable, discoverable, and verifiable task-specific capability. Use when creating or optimizing a skill, reviewing its scope or instructions, improving its description, or testing whether it adds value."
---

# Writing Good Skills

Write capability skills that improve agent behavior, not merely valid `SKILL.md` files.

A qualified capability skill turns a general agent's unstable performance on a coherent class of tasks into reusable, discoverable, and verifiable specialized capability. The best skill uses the least knowledge, process, and tooling that matches the task's fragility while producing the greatest counterfactual net benefit over no skill or the previous version.

## Start From A Capability Gap

Ground the skill in real evidence: execution traces, repeated corrections, domain artifacts, project conventions, failure cases, or successful task demonstrations. State what the target agent gets wrong, misses, or reinvents without the skill. If the agent already performs the target tasks reliably, do not create guidance for them.

Trace every candidate instruction or resource to an observed failure, an explicit requirement, or a discriminating evaluation case. Start with the smallest intervention that could change those outcomes. Keep plausible but unobserved neighboring risks out of the first version; do not turn a narrow failure set into a comprehensive domain handbook.

Define one coherent task family rather than one answer or a broad topic. Instances belong together when they need substantially the same specialized knowledge, decisions, tools, and success criteria. Split unrelated responsibilities; merge fragments that must be loaded together to complete one task.

See `references/judgment-dimensions.md` when the capability gap, task-family boundary, control level, or compatibility risk is unclear.

## Match Control To Fragility

Choose the instruction form independently for each part of the task:

| Task condition | Preferred form |
|---|---|
| Several approaches are valid and context determines the choice | principles, heuristics, and tradeoffs |
| A preferred pattern exists but variation is legitimate | a default with an explicit escape condition, template, or parameterized procedure |
| Steps are easy to omit or depend on prior state | a checklist with observable gates |
| Order or consistency determines correctness or safety | an exact workflow or tested script |
| Mechanical work is repeatedly reimplemented | a reusable script with explicit inputs, outputs, dependencies, and errors |

Explain why only when the reason helps the agent adapt a judgment to unseen cases or prevents it from rationalizing around a rule. Give direct instructions for fixed mechanical operations. Do not freeze a variable judgment task into one workflow, and do not leave a fragile operation to vague advice.

## Package Only Missing Knowledge

Include what changes behavior: domain procedures, project-specific facts, non-obvious gotchas, preferred tools, decision criteria, validation gates, examples, templates, or deterministic helpers. Omit generic explanations the target agent already handles.

Prefer one working default over a menu of equivalent options. Use examples to calibrate a rule or boundary, not to substitute for the reusable method. When existing wording, examples, or workflows induce unwanted behavior, remove that inducement before adding a counter-rule.

Use progressive disclosure:

- Keep the capability's core method and always-needed gotchas in `SKILL.md`.
- Put conditional detail and lookup material in focused `references/` files.
- Put reusable output material in `assets/` and deterministic operations in `scripts/`.
- Give every support-file pointer an observable read condition.

See `examples/progressive-disclosure-examples.md` when deciding what stays in `SKILL.md` or how to point to support files.

## Make The Skill Discoverable

Write the frontmatter `description` for the pre-load decision. It must identify both the capability and observable request, artifact, or task states that make the skill useful. Test realistic should-trigger prompts and near-miss should-not-trigger prompts; keyword overlap alone is not a valid boundary.

See `references/description-patterns.md` whenever creating or changing the description.

## Verify Counterfactual Net Benefit

Compare the skill against no skill or the previous version on realistic tasks in clean contexts. Include varied phrasings, unseen instances, edge cases, and adjacent tasks where the skill should remain inactive. Repeat runs when model variance could change the result.

Measure what the skill buys and costs: task success, output quality, trigger accuracy, harmful scope expansion, token use, elapsed time, and compatibility failures. Use executable checks for mechanical outcomes and blind or human review for qualities that cannot be reduced to assertions. Read traces to distinguish missing knowledge from ambiguous, excessive, or conflicting instructions.

Remove content that passes equally well without the skill. Add guidance only when a remaining failure identifies a missing decision, fact, gate, example, or deterministic operation. Revise or retire a skill that adds no meaningful benefit, costs more than it buys, or causes negative transfer. Do not call a skill qualified from format compliance or a plausible-looking draft alone.

See `references/evaluation-patterns.md` when behavior can be exercised or two versions need comparison.

## Completion Standard

Before finishing, confirm from the artifact and available evidence:

- A specific without-skill failure or unstable behavior justifies the skill.
- The scope is one reusable task family, not an instance answer or topic dump.
- Each instruction's freedom matches the fragility of the decision or operation.
- Every included rule, example, reference, or script supplies missing capability.
- Every included item traces to gap evidence, a requirement, or a discriminating test.
- The description separates intended triggers from realistic near misses.
- Evaluation shows net benefit, or the skill is explicitly marked unverified with the next discriminating test identified.
