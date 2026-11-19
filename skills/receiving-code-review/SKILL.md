---
name: receiving-code-review
description: "Evaluate existing code-review feedback before acting on it. Use when PR comments, inline annotations, pasted suggestions, or reviewer-agent findings exist and the task is deciding what to fix, escalate, or decline."
---

# Receiving Code Review

Filter review feedback through the current change's intent and risk before acting. A plausible comment is not automatically worth acting on, and a real issue is not automatically worth fixing in this change.

## Evaluate Against Evidence

Establish what the change is meant to achieve, preserve, and exclude from the issue or PR, conversation, tests, and code—do not invent intent.

For each item, check:

- **Consequence:** What materially breaks or degrades if unaddressed?
- **Relevance:** Does it affect the change's behavior, contracts, safety, verification, or regression risk?
- **Scope:** Was it introduced or exposed by this change, or is it pre-existing, stylistic, or speculative?
- **Proportionality:** Does the risk reduction justify the churn, complexity, and maintenance cost? Reject single-consumer abstractions and speculative future-proofing.

Verify load-bearing claims against actual callers, tests, and contracts; reviewer confidence, authority, or convergence is signal, not evidence. Separate a comment's **observation**, **premise**, and **proposed change**—accepting a concern does not require accepting the proposed remedy.

When feedback overlaps or conflicts, resolve higher-level correctness and scope issues before local edits they might discard, and group comments that share one underlying concern.

## Decide: Fix, Escalate, or Decline

- **Fix:** Relevant, consequential, and proportionate within current authorization.
- **Escalate:** Materially affects correctness, safety, contracts, or design intent (including goal conflicts that evidence cannot settle), but requires a scope, architecture, or product decision you are not authorized to make. Ask the smallest decision-changing question before related edits.
- **Decline:** Unsupported, preference-driven, speculative, disproportionate, or out of scope for this change. Do not modify code, add TODOs, or create follow-up work unless authorized. Surface a declined item to the user only when it is evidence-backed and could reasonably change a future decision—do not transfer low-value noise.

An independent security, data-loss, crash, or correctness risk still warrants **Fix** or **Escalate** even when pre-existing; do not use scope to hide a material risk.

## Output

For non-obvious items, report the disposition and load-bearing evidence plainly (no performative agreement). Draft reviewer-facing replies only when needed, and never imply follow-up work was created for declined items.
