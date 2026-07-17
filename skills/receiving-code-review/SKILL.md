---
name: receiving-code-review
description: "Judge existing code-review feedback by its value to the current change. Use when PR comments, inline annotations, pasted suggestions, screenshots, or reviewer-agent findings already exist and the task is deciding what should affect the current change, what needs user attention, and what should not be acted on."
---

# Receiving Code Review

Filter review feedback through the current change's intent and risk before acting. A technically plausible comment is not automatically valuable to the change, and a real issue is not automatically worth fixing now.

## Establish the Decision Frame

Identify what the change is intended to achieve, what behavior or contracts it must preserve, its stated non-goals, and the material risks it introduces or exposes. Use the conversation, issue or PR description, tests, code, and prior maintainer decisions as evidence. Do not invent intent that cannot be traced.

For each feedback item, ask only what is needed to decide:

- **Consequence:** What materially happens if this is not addressed?
- **Relevance:** Does it affect the change's intended behavior, contracts, safety, verification, or regression risk?
- **Scope:** Was it introduced or exposed by this change, or is it pre-existing, adjacent, stylistic, or speculative?
- **Proportionality:** Does the proposed remedy reduce a concrete risk enough to justify its churn, regression risk, complexity, and maintenance cost?

Verify load-bearing claims against repository evidence. Check actual callers, tests, supported platforms, and extension points before accepting claims about reuse, testability, compatibility, or future flexibility. Reviewer confidence, authority, or convergence is signal, not evidence.

When useful, separate the comment's **observation**, **premise**, and **proposed change**. One may be valid while the others are not. In particular, accepting a concern does not require accepting the proposed remedy.

## Choose a Disposition

Assign one current disposition:

- **Address now:** The item materially affects the change's intent or risk, and the response is proportionate. Proceed only within the user's existing authorization.
- **Escalate:** The item may materially affect correctness, safety, contracts, or design intent, but resolving it requires a scope, architecture, or product decision the agent is not authorized to make. Ask the smallest decision-changing question before related work.
- **Advisory:** The issue is evidence-backed and has a concrete consequence, but it does not block the current intent or is not proportionate to handle in this change. Surface it without modifying code, expanding scope, creating TODOs, or opening follow-up work unless the user authorizes that action.
- **Decline:** The item is preference-driven, speculative, unsupported, unrelated to meaningful risk, or has negative net value. Do not act on it. Explain only when the reviewer or user needs the decision.

An independent security, data-loss, crash, or correctness risk can require Address now or Escalate even when it predates the change. Do not use scope as a reason to hide a material risk.

Do not transfer every low-value comment to the user. Use Advisory only when a future decision could reasonably change because of the information; otherwise Decline it.

Read `references/evaluation-criteria.md` when relevance, scope, consequence, or proportionality is hard to distinguish. Read `references/multi-reviewer.md` when comments overlap, contradict each other, or a higher-level decision may invalidate lower-level feedback. Read `references/response-language.md` only when drafting reviewer-facing replies.

## Output

For non-obvious items, report the **Disposition** and the load-bearing **Basis**. For Advisory, state the finding, why it is not being handled now, and the concrete consequence. Draft a reviewer-facing response only when one is needed.
