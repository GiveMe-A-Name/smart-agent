---
name: technical-communication
description: "Improve persistent technical artifacts so later readers can understand, decide, review, or operate without author context. TRIGGER when writing or revising design docs, RFCs, ADRs, postmortems, runbooks, READMEs, API docs, PR descriptions, or commit messages. DO NOT TRIGGER for implementation planning, isolated inline comments, or ephemeral messages."
---

# Technical Communication

Write durable technical artifacts that a later reader can use without the author present. Optimize for the reader's next action: decide, review, implement, operate, integrate, or understand.

## Principles

- **Start from reader action.** Before drafting or revising, identify the artifact's primary reader and the action they need to take from the user request, source artifact, or conversation evidence [because audience and action determine structure, detail level, vocabulary, and what evidence must appear].
- **Choose the vehicle from the job.** Use the document type that matches the action: RFC/design doc for evaluating a proposal, ADR for recording a decision, postmortem for preventing recurrence, runbook for operating under pressure, README/API docs for using a system, and PR description or commit message for preserving change intent.
- **Put the entry point first.** Decision, proposal, review, and change-summary artifacts start with the recommendation, change intent, or ask. Operable and reference artifacts start with when to use the artifact, prerequisites, and the first check or command [because many readers act on the first usable instruction].
- **Write for scanning, not linear reading.** Headings must name reader questions or tasks. Use tables for option comparisons and multi-dimensional tradeoffs; use diagrams for relationships, flows, state transitions, or dependency graphs; use code examples for concrete API behavior.
- **Separate facts, analysis, and recommendations.** A reader must be able to challenge the analysis without disputing the facts, or accept the facts while rejecting the recommendation.
- **Anchor load-bearing claims.** Any claim that justifies a decision, priority, risk, urgency, or correctness needs a number, concrete condition, example, link, observed evidence, or explicit assumption. Remove or qualify claims that have no available anchor.
- **Match detail to the reader.** Approval readers need outcomes, tradeoffs, and risks. Implementers need file paths, contracts, commands, examples, and edge cases. Mixed audiences need layered structure rather than averaged detail.
- **Preserve context the diff cannot carry.** PR descriptions and commit messages should explain why the change exists, what risk matters, what alternatives or constraints shaped it, and where future readers should look for source context.

## Constraints

Keep these constraints true while editing a persistent artifact:

- Do not leave vague urgency words such as "soon", "often", "sometimes", "various", or "may" in claims that drive action; replace them with a condition, frequency, number, example, or explicit uncertainty.
- Do not document implementation internals as an API contract. API docs must state inputs, outputs, errors, behavior, and compatibility expectations that callers depend on.
- Do not treat code comments as technical-communication work unless they are part of a broader documentation artifact or documentation pass. In that case, comments explain why a non-obvious choice exists, not what the code already says.
- Do not update point-in-time artifacts such as ADRs or postmortems as if they were living docs; supersede them when the decision or understanding changes.
- Do not publish or materially revise a living artifact without an update path: owner, source of truth, review trigger, repository convention, or explicit assumption.

## Artifact Patterns

Use these patterns as the first structural choice:

- **RFC / Design Doc:** State the proposed change, decision needed, alternatives, tradeoffs, non-goals, rollout or migration concerns, and review focus.
- **ADR:** Record the decision, context, constraints, alternatives considered, consequences, and supersession path. Treat the artifact as immutable.
- **Postmortem:** Separate impact, timeline, root causes, contributing factors, and follow-up actions. Actions need owners and concrete completion conditions.
- **Runbook:** Start from symptoms and prerequisites, then exact checks, commands, expected outputs, rollback or mitigation steps, and escalation criteria.
- **README:** Help a new developer or consumer reach a working state: what this is, prerequisites, setup, run/use commands, common failure modes, and where to find maintained references.
- **API Docs:** Document the caller contract: valid inputs, outputs, errors, side effects, compatibility, examples, and versioning or deprecation behavior.
- **PR Description:** Provide context the diff cannot: motivation, risk, test evidence, rollout concerns, and where reviewers should focus.
- **Commit Message:** Use one line only when the subject explains the change and why it exists. Add a body when future readers would ask "why", "why now", or "what changed semantically?"

See `references/judgment-dimensions.md` when a document needs deeper judgment on audience, vehicle, structure, precision, longevity, code documentation, everyday technical writing, or visual communication.
See `examples/artifact-examples.md` for calibrated positive and negative examples.
