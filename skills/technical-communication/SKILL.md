---
name: technical-communication
description: "Invoke when writing technical documents that will be read by others — design docs, RFCs, ADRs, postmortems, runbooks, migration guides, or any document that records a technical decision or enables someone else to act. Cost of unnecessary invocation: a short writing quality pass. Cost of missing: documents that are ignored because they are unreadable, decisions that are relitigated because they were never properly recorded, or knowledge that is lost because it lived only in someone's head. When uncertain whether the writing quality matters, invoke."
---

# Technical Communication

Write technical documents that are read, understood, and acted upon. The value of a technical document is not in writing it — it is in the decisions it enables, the knowledge it preserves, and the alignment it creates.

## Trigger Logic

**Invocation default**: A technical document that nobody reads has negative value — it cost time to write and creates a false sense that the information has been communicated. The cost of applying writing discipline is a short review pass. The cost of skipping it is wasted effort, unread documents, and decisions that are relitigated because they were never properly recorded. Invoke when writing any document intended for others to read, understand, or act upon.

Do not use when:
- writing code comments (that is part of implementation judgment)
- the communication is informal and ephemeral — a Slack message, verbal discussion, or scratch note

## Capability Boundary

This skill owns:
- judging who the audience is and what they need from this document
- choosing the right document vehicle for the communication need
- structuring technical documents for clarity and impact
- judging the appropriate level of detail for the audience
- evaluating whether the document will remain useful over time

This skill does not own:
- the technical content itself — it teaches how to communicate, not what to communicate
- document tooling or formatting systems
- organizational document approval processes

## Invariants

- The reader's context, not the author's, determines the right level of detail. Never default to "the reader should know this."
- The conclusion or recommendation must appear in the first paragraph. Reasoning and evidence follow; they do not precede.
- Every sentence must help the reader understand, decide, or act. Delete anything that exists only to demonstrate thoroughness.
- Every living document (runbook, API docs, architecture overview) must name an update owner before publishing. A document without an owner will eventually lie.

## Decision Signals

Work through these dimensions for each document. The first two must be answered before writing begins. See `references/judgment-dimensions.md` for detailed signals in each dimension.

**Audience** — Who reads this and what action do they need to take?
- Decision-makers need outcomes, tradeoffs, and risks — not implementation steps.
- Implementation teams need specifics: file paths, code examples, exact commands.
- Mixed audiences: executive summary up front, detail below — do not average the level.

**Vehicle** — Which document type fits this need?
- RFC/Design Doc: proposing a change before building it; must include alternatives and non-goals
- ADR: recording a decision with the context that made it right at the time; treat as immutable
- Postmortem: root cause, not proximate trigger; action items with owners and deadlines
- Runbook: operable at 3am with no prior context; symptoms first, then commands

**Structure** — Can a reader get the key point without reading past the first paragraph?
- If the conclusion is not in paragraph 1, move it there before anything else.
- Headings are navigation tools — "Background" is a label; "Why current auth cannot support SSO" is useful.
- Facts, analysis, and recommendations must be separated so readers can evaluate each independently.

**Precision** — Is every claim specific enough to act on?
- Replace "sometimes", "often", "various", "may" with specific numbers, frequencies, and conditions.
- Use tables for comparisons — a table across three options is processed in 10 seconds; three paragraphs take 3 minutes.
- Show examples and numbers rather than assertions — evidence makes writing actionable.

**Longevity** — Is the update expectation explicit?
- Point-in-time documents (ADRs, postmortems): supersede rather than update; do not append corrections.
- Living documents: state explicitly who owns updates and what triggers a review.

For code documentation, PR descriptions, commit messages, and visual communication, see `references/judgment-dimensions.md`.

## Failure Signals

Stop and reassess if:

- you cannot name who will read this document before you start writing
- the recommendation or conclusion is not in the first paragraph
- you are writing implementation detail for a leadership audience, or high-level summaries for the implementation team
- the document uses jargon the audience will not know without explanation
- claims use "sometimes", "often", "various", or "may" without specific supporting data
- alternatives were not considered or not documented for a decision document
- the document describes a living system but has no named update owner
- PR descriptions and commit messages lack the context the diff cannot provide — the "why" is missing
- code documentation fails its job: README can't reach a running state, API docs describe implementation instead of contract, or comments restate the code instead of explaining why

## Completion Criteria

- [ ] The audience was identified and the right vehicle was chosen before writing began.
- [ ] The conclusion or recommendation appears in the first paragraph.
- [ ] Every claim is specific — numbers, examples, or evidence rather than assertions.
- [ ] The document is structured for scanning, not linear reading.
- [ ] Update ownership is explicit: immutable (supersede to change) or named owner for living documents.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the document is genuinely useful.

Am I confusing polished prose with reader usefulness?

Am I exiting because the document is genuinely clear and useful, or because it now reads smoothly enough to me?
