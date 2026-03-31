---
name: technical-communication
description: "Invoke when writing technical documents that will be read by others — design docs, RFCs, ADRs, postmortems, runbooks, migration guides, or any document that records a technical decision or enables someone else to act. Cost of unnecessary invocation: a short writing quality pass. Cost of missing: documents that are ignored because they are unreadable, decisions that are relitigated because they were never properly recorded, or knowledge that is lost because it lived only in someone's head. When uncertain whether the writing quality matters, invoke."
---

# Technical Communication

Write technical documents that are read, understood, and acted upon. The value of a technical document is not in writing it — it is in the decisions it enables, the knowledge it preserves, and the alignment it creates.

Most technical documents fail not because the technical content is wrong, but because they are poorly structured, too long, or written for the author instead of the reader. This skill teaches the judgment to write documents that work.

## Trigger Logic

**Invocation default**: A technical document that nobody reads has negative value — it cost time to write and creates a false sense that the information has been communicated. The cost of applying writing discipline is a short review pass. The cost of skipping it is wasted effort, unread documents, and decisions that are relitigated because they were never properly recorded. Invoke when writing any document intended for others.

Use when:
- writing a design document, RFC, or technical proposal
- recording a technical decision (ADR)
- writing a postmortem or incident review
- creating a runbook or operational guide
- writing documentation for a system, API, or process
- communicating technical information to non-technical stakeholders

Do not use when:
- writing code comments (that is part of implementation judgment)
- writing commit messages (that is part of the git workflow)
- the communication is informal and ephemeral (Slack message, verbal discussion)

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

- Write for the reader, not for yourself. The reader's context, not yours, determines the right level of detail.
- State the conclusion first. Technical readers want the answer before the reasoning.
- Every document must answer: what is this, why does it matter, what do I need to do?
- Shorter is better, until it becomes unclear. Then it's too short.
- A document that is never updated is a document that will eventually lie. Build in update expectations or mark documents as point-in-time.
- **Before exiting this skill, you MUST complete the Self-Check section at the end.**

---

## Reading the Situation

When facing a communication task, use this table to identify which dimensions need the most attention:

| Communication need | Primary dimension | Also check |
|---|---|---|
| Proposing a technical change | Vehicle Selection (→ RFC) | Audience Awareness, Structure for Impact |
| Recording a decision already made | Vehicle Selection (→ ADR) | Longevity, Precision |
| Responding to an incident | Vehicle Selection (→ Postmortem) | Precision, Structure for Impact |
| Enabling operations/troubleshooting | Vehicle Selection (→ Runbook) | Audience Awareness, Precision |
| Explaining a system to a new team | Audience Awareness | Structure for Impact, Longevity |
| Communicating to leadership | Audience Awareness | Structure for Impact, Precision |
| Writing docs that keep going stale | Longevity | Audience Awareness |
| Document exists but nobody reads it | Structure for Impact | Audience Awareness, Precision |
| Feedback that document is "too vague" | Precision and Conciseness | Audience Awareness |

---

## Judgment Dimensions

When writing or reviewing a technical document, work through these dimensions. Not every dimension applies to every document — but skipping a relevant one is how documents fail silently.

### 1. Audience Awareness

**Question**: Who will read this, and what do they need from it?

**Key signals**:
- If you cannot name the audience, the document will be written for yourself. Stop and identify who will read it before writing.
- If the audience includes people with decision-making authority, they need outcomes, tradeoffs, and risks — not implementation steps. If they need to approve something, the document must make the decision easy, not demonstrate your thoroughness.
- If the audience is the implementation team, they need specifics: file paths, code examples, interface contracts, exact commands. High-level summaries waste their time.
- If the audience spans multiple levels (e.g., both leadership and engineers), use layered structure: executive summary up front for decision-makers, detailed sections below for implementers. Do not average the detail level — serve both.
- If the audience includes people outside your domain, every piece of jargon is a barrier. Each unexplained term is a point where a reader stops trusting the document and starts skimming.
- The action the reader needs to take determines what to include. If they need to approve: give them the tradeoff. If they need to implement: give them the spec. If they need to understand: give them the context. If they need to operate: give them the commands.

### 2. Vehicle Selection

**Question**: What is the right document type for this communication need?

**Key signals**:
- If the goal is to **propose a change and get feedback before building**, the vehicle is an **RFC / Design Doc**. The document succeeds when reviewers can evaluate the approach and alternatives without asking clarifying questions. It fails when alternatives are missing (reviewers wonder if other options were considered), when there are no explicit non-goals (scope creep during review), or when there is too much implementation detail (locks in decisions prematurely).
- If the goal is to **record a decision that was made, with the context that informed it**, the vehicle is an **ADR**. The document succeeds when someone 6 months from now can read it and understand *why* the decision was made — not just *what* was decided. ADRs are immutable: when a decision changes, a new ADR supersedes the old one. It fails when context is omitted ("we chose X" without explaining the forces that made X the right choice at that time).
- If the goal is to **learn from an incident to prevent recurrence**, the vehicle is a **Postmortem**. The document succeeds when it identifies root causes (not just triggers), assigns specific and time-bound action items with owners, and treats the incident as a system failure rather than an individual failure. It fails when it stops at the proximate cause ("the config was wrong" instead of "config changes are not validated before deployment"), when action items have no owners, or when blame language discourages honest reporting next time.
- If the goal is to **enable someone to operate or troubleshoot a system**, the vehicle is a **Runbook**. The document succeeds when a tired on-call engineer at 3am can follow it without prior knowledge of the system. It fails when it describes what to do instead of giving exact commands, when it starts with causes instead of symptoms, or when it omits escalation criteria.
- If the goal does not map to any of these, ask: is this document recording a decision, enabling an action, or preserving knowledge? The answer determines the structure even if the label is informal.

### 3. Structure for Impact

**Question**: How should I structure this for maximum clarity and ease of use?

**Key signals**:
- If the conclusion or recommendation is not in the first paragraph, most readers will never reach it. State the key point first. The rest of the document supports and justifies that point. If a reader stops after the first paragraph, they should still have the essential information.
- If readers will need different parts of the document, structure for scanning, not linear reading. Headings are navigation — "Background" is weak; "Why the current auth system can't support SSO" is useful. A reader should be able to find their section from the heading alone.
- If you are comparing options, alternatives, or attributes, use a table. A table comparing three options is processed in 10 seconds; three paragraphs describing the same options take 3 minutes.
- If a paragraph covers two topics, split it. The test: if a reader highlights the paragraph, would they highlight all of it or just half?
- If facts, analysis, and recommendations are mixed together, readers cannot evaluate the reasoning. Separate them explicitly. "The service handles 10,000 rps" is a fact. "At current growth, we exceed the pool limit in 3 months" is analysis. "We recommend increasing the pool now" is a recommendation. When these are separated, readers can agree with the facts while questioning the recommendation — which is exactly how good technical decisions are made.

### 4. Precision and Conciseness

**Question**: Am I being specific enough, and concise enough?

**Key signals**:
- If a statement contains "sometimes," "often," "various," "some," or "may," it is probably too vague to be actionable. "The service sometimes returns errors" tells the reader nothing. "The payment service returns 503 errors ~50 times/day during peak hours (2-4pm UTC), affecting ~0.1% of checkout attempts" tells them everything.
- If a paragraph can be replaced by a bullet list or a table without losing meaning, it should be. Prose is for reasoning and narrative; structured formats are for data and comparisons.
- If you are explaining something that could be shown with a number, a diagram, or an example, use the number, diagram, or example. Evidence makes writing actionable; assertions make writing ignorable.
- If a sentence does not help the reader understand, decide, or act, delete it. Context is valuable only when the reader needs it. Background sections that exist because "the reader should know this" rather than "the reader needs this to understand the next section" are padding.
- If the document is long, ask: is it long because the topic is complex, or because the writing is not tight? Complex topics justify length. Repetition, throat-clearing, and unnecessary context do not.

### 5. Longevity

**Question**: Will this document still be useful in 6 months? Should it be?

**Key signals**:
- If the document records a decision (ADR) or an incident (postmortem), it should be treated as immutable — a snapshot of understanding at a point in time. Do not update it; supersede it. Adding "Update (March 2025): we changed this because..." corrupts the historical record.
- If the document describes a living system (runbook, API docs, architecture overview), it *will* become stale. Build in explicit update expectations: who owns updates, what triggers a review, where the source of truth lives. A document without an update owner is a document that will eventually lie.
- If the document references specific versions, dates, people, or system states, mark it as point-in-time or ensure those references will be maintained. "The current architecture" becomes meaningless in 6 months without a date or version.
- If the document is a proposal (RFC), state its expected lifecycle: does it become an ADR when accepted? Does it become obsolete once implemented? Documents with unclear lifecycles accumulate and confuse.
- If you're unsure whether the document should be updated or replaced, default to immutable with supersession. It is easier to write a new document with current context than to correctly update an old document without introducing contradictions.

---

## Failure Signals

Stop and reassess if:
- you cannot name the audience — you don't know who will read this document
- the recommendation or conclusion is buried after pages of context
- the document uses jargon the audience won't know without explanation
- you are writing implementation details for a leadership audience or high-level summaries for the implementation team
- the document has no structure — it reads as a stream of consciousness
- alternatives were not considered or not documented (for decision documents)
- the document makes claims without evidence or specifics
- you are writing a document that duplicates information already documented elsewhere
- the document has no update owner and describes a living system
- you chose a document type out of habit rather than by matching it to the communication need

## Self-Check Before Exiting

- [ ] Did I identify the audience and what they need? (Dimension 1)
- [ ] Did I choose the right vehicle for this communication need? (Dimension 2)
- [ ] Does the document state its conclusion in the first paragraph and structure for scanning? (Dimension 3)
- [ ] Is the writing specific and concise — numbers over adjectives, evidence over assertions? (Dimension 4)
- [ ] Have I addressed whether this document should be updated or treated as immutable? (Dimension 5)
- [ ] Am I exiting because the document is genuinely clear and useful, or rationalizing?

**If any check fails, return to the relevant dimension before exiting.**
