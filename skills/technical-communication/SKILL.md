---
name: technical-communication
description: "Write durable technical communication artifacts that others will read, decide from, review from, or operate from — design docs, RFCs, ADRs, postmortems, runbooks, READMEs, API docs, PR descriptions. TRIGGER when: writing a persistent artifact that must carry context without the author present. DO NOT TRIGGER when: writing inline code comments or informal, ephemeral messages."
---

# Technical Communication

Write durable technical communication artifacts that are read, understood, and acted upon. The value of a technical artifact is not in writing it — it is in the decisions it enables, the knowledge it preserves, and the alignment it creates when the author is not present to explain it.

## Trigger Logic

**Invocation default**: A durable technical artifact that nobody can use has negative value — it cost time to write and creates a false sense that the information has been communicated. The cost of applying writing discipline is a short review pass. The cost of skipping it is wasted effort, unread documents, and decisions that are relitigated because the artifact did not carry the needed context. Invoke when writing a persistent artifact that another person must read later to understand, decide, review, or operate without relying on you to fill in missing context.

Do not use when:
- writing inline code comments inside implementation
- the communication is informal and ephemeral — a Slack message, verbal discussion, or scratch note

## Capability Boundary

This skill owns:
- judging who the audience is and what they need from this document
- choosing the right document vehicle for the communication need
- structuring persistent technical artifacts for clarity and impact
- judging the appropriate level of detail for the audience
- evaluating whether the artifact will remain useful over time

This skill does not own:
- the technical content itself — it teaches how to communicate, not what to communicate
- document tooling or formatting systems
- organizational document approval processes

## Invariants

- Before drafting body text, name the primary reader and the action they need to take. If either is missing, stop.
- Decision, proposal, review, and change-summary artifacts must state the decision, recommendation, or change intent in the first paragraph. Operable and reference artifacts must begin with an opening section that states when to use the artifact, required prerequisites, and the first check or command.
- Any claim used to justify a decision, priority, risk, or action must include at least one supporting anchor: a number, a concrete condition, an example, a link, or observed evidence. If it has none, mark it as an assumption or revise it.
- Any artifact expected to remain current as the system, interface, or operating procedure changes is a living artifact. Every living artifact (runbook, API docs, architecture overview, README) must name an update owner or source of truth before publishing.

## Decision Signals

Work through these dimensions for each artifact. The first two must be answered before writing begins. See `references/judgment-dimensions.md` for detailed signals in each dimension.

**Audience** — Who reads this and what action do they need to take?
- If you cannot name the primary reader and their required action in one sentence, stop before drafting.
- If the primary reader approves or rejects rather than implements, the opening must contain outcomes, tradeoffs, and risks, not step-by-step implementation.
- If the primary reader will implement or operate the change, include the concrete material they act on: file paths, code examples, interface contracts, or exact commands.
- Mixed audiences require layered structure: a summary section before implementation detail.

**Vehicle** — Which document type fits this need?
- RFC/Design Doc: proposing a change before building it; must include alternatives and non-goals
- ADR: recording a decision with the context that made it right at the time; treat as immutable
- Postmortem: root cause, not proximate trigger; action items with owners and deadlines
- Runbook: operable at 3am with no prior context; symptoms first, then commands
- README/API docs: help someone use or integrate with a system without reading the implementation
- PR description/commit message: explain intent, risk, and context that the diff alone cannot carry

**Structure** — Can a reader get the right entry point without reading linearly?
- Decision/proposal/review artifacts: paragraph 1 states the recommendation, change intent, or ask.
- Operable/reference artifacts: the opening section states when to use this artifact, what prerequisites apply, and the first check or command.
- Headings are navigation tools. Use headings that name a question, decision, or task a reader would search for; avoid filler headings such as "Background" when a more specific heading is available.
- When disagreement is plausible, separate facts, analysis, and recommendations into distinct paragraphs or sections.

**Precision** — Is every claim specific enough to act on?
- Replace "sometimes", "often", "various", "may" with specific numbers, frequencies, and conditions.
- Comparisons across three or more options or two or more dimensions should use a table unless a stronger format is clearly better.
- Claims that justify urgency, risk, or correctness need evidence: a number, example, condition, link, or explicit assumption label.

**Longevity** — Is the update expectation explicit?
- Point-in-time documents (ADRs, postmortems): supersede rather than update; do not append corrections.
- Living artifacts are expected to stay current after publication; they must state who owns updates or where the maintained source of truth lives.

For code documentation, PR descriptions, commit messages, and visual communication, see `references/judgment-dimensions.md`.

## Failure Signals

Stop and reassess if:

- you cannot name the primary reader and their required action before drafting
- a decision/proposal/review artifact reaches paragraph 2 without stating the recommendation, change intent, or ask
- an operable/reference artifact begins without an opening section stating when to use it, prerequisites, or the first check or command
- you are writing implementation detail for a leadership audience, or high-level summaries for the implementation team
- the artifact uses jargon the primary reader cannot be expected to know and does not define it on first use
- a claim driving urgency, risk, or correctness has no number, example, condition, link, observed evidence, or explicit assumption label
- alternatives were not considered or not documented for a decision artifact
- the artifact is expected to stay current after publication but has no named update owner or source of truth
- a PR description or commit message lacks the context the diff cannot provide
- code documentation fails its job: README cannot reach a running state, API docs describe implementation instead of contract, or an inline comment only restates the code

## Completion Criteria

- [ ] The primary reader and their required action were named before drafting, and the vehicle matches that need.
- [ ] The opening follows the artifact type rule: recommendation/change intent in paragraph 1 for decision/proposal/review artifacts, or an opening section with entry conditions plus first check/command for operable/reference artifacts.
- [ ] Claims that justify urgency, risk, correctness, or priority include evidence or are explicitly labeled as assumptions.
- [ ] The artifact is structured for retrieval, not linear reading: headings name questions, decisions, or tasks readers will look for.
- [ ] Update handling is explicit: immutable artifacts are superseded to change; living artifacts name an update owner or source of truth.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the document is genuinely useful.

Am I confusing polished prose with reader usefulness?

Am I exiting because the document is genuinely clear and useful, or because it now reads smoothly enough to me?
