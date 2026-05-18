---
name: receiving-code-review
description: "Evaluate existing review feedback before action. TRIGGER when PR comments, inline annotations, pasted suggestions, screenshots, or reviewer-agent output exist and the task is deciding whether to accept, push back, clarify, or implement them. DO NOT TRIGGER when reviewing code changes directly, requesting a review, or applying feedback that has already been evaluated and accepted."
---

# Receiving Code Review

Evaluate review feedback as claims, not instructions. A reviewer comment usually contains three separable parts: an **observation** about what the code does, a **premise** about what the code should optimize for, and a **proposed change**. Verify each part independently before acting.

Use three evidence sources: explicit intent, repository evidence, and architectural constraints. Explicit intent can come from the current conversation, PR description, issue, design doc, commit message, code comments, tests, or prior user or maintainer decisions. If intent cannot be traced, mark intent unknown and clarify instead of inventing it.

## Principles

- Reviewer authority is not evidence. Human, automated, and agent-written reviews all require verification against the codebase and stated intent.
- A correct observation can still lead to a wrong premise or harmful proposed change.
- Collaboration is not compliance. Accept, push back, and clarify are all valid outcomes when they are evidence-backed.
- Clarification is a decision when the missing answer changes whether the fix applies, where it lands, or which contract it touches.
- Multiple reviewers converging on an issue increases the signal, but does not replace evidence.
- Do not use performative agreement or gratitude as a substitute for evaluation. State the decision and the evidence.

## Constraints

- Assign each feedback item exactly one current decision: **Accept**, **Push back**, or **Clarify**.
- Before accepting a non-mechanical suggestion, cite the code, test, usage, or platform evidence that makes the observation technically correct.
- For design or architecture feedback, cite the explicit intent source used to evaluate the premise. If no intent source exists, choose Clarify unless the item is an independent bug, security issue, or correctness problem.
- Accept only when the suggestion is technically correct and aligned with intent, identifies a bug or security issue regardless of intent, or improves clarity without changing behavior.
- Push back when the suggestion breaks existing behavior, contradicts established intent, adds unsupported generality, violates repository constraints, or is technically wrong for this stack.
- Pushback must name the reviewer's premise and the contradicting evidence or constraint. Do not use "I prefer it this way" as the reason.
- Clarify when the suggestion has multiple plausible interpretations, requires a design-intent change, or depends on context that is not present. The question must name the competing interpretations and the implementation choice that depends on the answer.
- If one feedback item depends on an unresolved clarification, mark it blocked or dependent. Independent bug fixes can proceed only when they remain correct under every plausible clarification outcome.
- For accepted non-trivial changes, decide the landing shape before editing code: responsibility boundary, interface impact, contract impact, and test coverage. Mechanical fixes can proceed directly.
- Do not write "you're right", "great point", "thanks", or equivalent agreement-signaling. Use direct language such as "Fixed by...", "I will change...", "I do not think this should change because...", or "I need to clarify...".

## Decision Checks

Use `references/evaluation-criteria.md` when a comment mixes problem, preference, scope, and value claims.

- **Bug, logic, security, data loss**: verify against code and tests; these can be valid even when intent is unclear.
- **Design or architecture**: compare the premise with explicit intent and local boundaries. A different architecture is not automatically better.
- **Style or naming**: author's call unless a style guide, local convention, or public API consistency requires the change.
- **Over-engineering or testability**: grep for actual call sites, tests, and extension points before adding abstraction. If no usage evidence exists, the benefit is hypothetical.
- **Reviewer confusion**: improve code clarity when the confusion is reasonable, even if the proposed fix is wrong.

Apply the claim split to common review shapes:
- "Extract this for testability" means verify current tests, likely callers, and whether the abstraction would actually be exercised.
- "This should live in middleware" means verify the current error-handling contract and whether middleware would change caller-visible behavior.
- "This does not support X" means verify whether the code supports X, then separately verify whether X is an explicit goal.

## Multiple Reviewers

Use `references/multi-reviewer.md` when feedback volume, overlap, or contradiction changes the decision process.

Triage before processing linearly when comments span multiple severity levels or can invalidate each other:
1. Blocking correctness, security, data loss, or crash risks.
2. Design-level feedback that can change whether the approach should exist or where the work belongs.
3. Implementation feedback such as error handling, local correctness, tests, and maintainability.
4. Style and preference feedback.

Surface contradictions instead of silently choosing one reviewer. Identify the goal each reviewer optimizes for, check whether a synthesis satisfies both goals, and escalate only when the goals conflict or intent authority is needed. If design-level rework is accepted, do not spend time fixing style in code that will be replaced.

## Output

For each feedback item, produce:
- **Decision**: Accept, Push back, or Clarify.
- **Evidence**: code, test, usage, platform, or intent source used for the decision.
- **Reasoning**: why the observation, premise, and proposed change are valid or invalid.
- **Response**: the concise reviewer-facing wording, when a response is needed.

Use `references/response-language.md` when drafting reviewer-facing replies. Reply in the original thread when the review system supports inline replies.
