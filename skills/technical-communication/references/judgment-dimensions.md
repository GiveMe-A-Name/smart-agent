# Judgment Dimensions — Detailed Reference

Deep signals for each writing dimension. Reach for this when a dimension needs more than the condensed guidance in SKILL.md.

---

## Choosing the Right Dimension

When facing a communication task, use this table to identify which dimensions need the most attention:

| Communication need | Primary dimension | Also check |
|---|---|---|
| Proposing a technical change | Vehicle Selection (→ RFC) | Audience Awareness, Structure |
| Recording a decision already made | Vehicle Selection (→ ADR) | Longevity, Precision |
| Responding to an incident | Vehicle Selection (→ Postmortem) | Precision, Structure |
| Enabling operations/troubleshooting | Vehicle Selection (→ Runbook) | Audience Awareness, Precision |
| Explaining a system to a new team | Audience Awareness | Structure, Longevity |
| Communicating to leadership | Audience Awareness | Structure, Precision |
| Writing docs that keep going stale | Longevity | Audience Awareness |
| Document exists but nobody reads it | Structure | Audience Awareness, Precision |
| Feedback that document is "too vague" | Precision | Audience Awareness |
| Writing a PR description | Everyday Technical Writing | Audience Awareness, Precision |
| Writing commit messages | Everyday Technical Writing | Precision |
| Writing a README for a module or project | Code Documentation | Audience Awareness, Longevity |
| Writing API documentation | Code Documentation | Precision, Longevity |
| Explaining tech debt to non-technical stakeholders | Audience Awareness | Structure, Precision |
| Choosing between text and a diagram | Visual Communication | Structure |

---

## 1. Audience Awareness

**Question**: Who will read this, and what do they need from it?

- If you cannot name the audience, the document will be written for yourself. Stop and identify who will read it before writing.
- If the audience includes people with decision-making authority, they need outcomes, tradeoffs, and risks — not implementation steps. If they need to approve something, the document must make the decision easy, not demonstrate your thoroughness.
- If the audience is the implementation team, they need specifics: file paths, code examples, interface contracts, exact commands. High-level summaries waste their time.
- If the audience spans multiple levels (e.g., both leadership and engineers), use layered structure: executive summary up front for decision-makers, detailed sections below for implementers. Do not average the detail level — serve both.
- If the audience includes people outside your domain, every piece of jargon is a barrier. Each unexplained term is a point where a reader stops trusting the document and starts skimming.
- The action the reader needs to take determines what to include. If they need to approve: give them the tradeoff. If they need to implement: give them the spec. If they need to understand: give them the context. If they need to operate: give them the commands.

---

## 2. Vehicle Selection

**Question**: What is the right document type for this communication need?

- If the goal is to **propose a change and get feedback before building**, the vehicle is an **RFC / Design Doc**. The document succeeds when reviewers can evaluate the approach and alternatives without asking clarifying questions. It fails when alternatives are missing (reviewers wonder if other options were considered), when there are no explicit non-goals (scope creep during review), or when there is too much implementation detail (locks in decisions prematurely).
- If the goal is to **record a decision that was made, with the context that informed it**, the vehicle is an **ADR**. The document succeeds when someone 6 months from now can read it and understand *why* the decision was made — not just *what* was decided. ADRs are immutable: when a decision changes, a new ADR supersedes the old one. It fails when context is omitted ("we chose X" without explaining the forces that made X the right choice at that time).
- If the goal is to **learn from an incident to prevent recurrence**, the vehicle is a **Postmortem**. The document succeeds when it identifies root causes (not just triggers), assigns specific and time-bound action items with owners, and treats the incident as a system failure rather than an individual failure. It fails when it stops at the proximate cause ("the config was wrong" instead of "config changes are not validated before deployment"), when action items have no owners, or when blame language discourages honest reporting next time.
- If the goal is to **enable someone to operate or troubleshoot a system**, the vehicle is a **Runbook**. The document succeeds when a tired on-call engineer at 3am can follow it without prior knowledge of the system. It fails when it describes what to do instead of giving exact commands, when it starts with causes instead of symptoms, or when it omits escalation criteria.
- If the goal does not map to any of these, ask: is this document recording a decision, enabling an action, or preserving knowledge? The answer determines the structure even if the label is informal.

---

## 3. Structure for Impact

**Question**: How should I structure this for maximum clarity and ease of use?

- If the conclusion or recommendation is not in the first paragraph, most readers will never reach it. State the key point first. The rest of the document supports and justifies that point. If a reader stops after the first paragraph, they should still have the essential information.
- If readers will need different parts of the document, structure for scanning, not linear reading. Headings are navigation — "Background" is weak; "Why the current auth system can't support SSO" is useful. A reader should be able to find their section from the heading alone.
- If you are comparing options, alternatives, or attributes, use a table. A table comparing three options is processed in 10 seconds; three paragraphs describing the same options take 3 minutes.
- If a paragraph covers two topics, split it. The test: if a reader highlights the paragraph, would they highlight all of it or just half?
- If facts, analysis, and recommendations are mixed together, readers cannot evaluate the reasoning. Separate them explicitly. "The service handles 10,000 rps" is a fact. "At current growth, we exceed the pool limit in 3 months" is analysis. "We recommend increasing the pool now" is a recommendation. When these are separated, readers can agree with the facts while questioning the recommendation — which is exactly how good technical decisions are made.

---

## 4. Precision and Conciseness

**Question**: Am I being specific enough, and concise enough?

- If a statement contains "sometimes," "often," "various," "some," or "may," it is probably too vague to be actionable. "The service sometimes returns errors" tells the reader nothing. "The payment service returns 503 errors ~50 times/day during peak hours (2-4pm UTC), affecting ~0.1% of checkout attempts" tells them everything.
- If a paragraph can be replaced by a bullet list or a table without losing meaning, it should be. Prose is for reasoning and narrative; structured formats are for data and comparisons.
- If you are explaining something that could be shown with a number, a diagram, or an example, use the number, diagram, or example. Evidence makes writing actionable; assertions make writing ignorable.
- If a sentence does not help the reader understand, decide, or act, delete it. Context is valuable only when the reader needs it. Background sections that exist because "the reader should know this" rather than "the reader needs this to understand the next section" are padding.
- If the document is long, ask: is it long because the topic is complex, or because the writing is not tight? Complex topics justify length. Repetition, throat-clearing, and unnecessary context do not.

---

## 5. Longevity

**Question**: Will this document still be useful in 6 months? Should it be?

- If the document records a decision (ADR) or an incident (postmortem), it should be treated as immutable — a snapshot of understanding at a point in time. Do not update it; supersede it. Adding "Update (March 2025): we changed this because..." corrupts the historical record.
- If the document describes a living system (runbook, API docs, architecture overview), it *will* become stale. Build in explicit update expectations: who owns updates, what triggers a review, where the source of truth lives. A document without an update owner is a document that will eventually lie.
- If the document references specific versions, dates, people, or system states, mark it as point-in-time or ensure those references will be maintained. "The current architecture" becomes meaningless in 6 months without a date or version.
- If the document is a proposal (RFC), state its expected lifecycle: does it become an ADR when accepted? Does it become obsolete once implemented? Documents with unclear lifecycles accumulate and confuse.
- If you're unsure whether the document should be updated or replaced, default to immutable with supersession. It is easier to write a new document with current context than to correctly update an old document without introducing contradictions.

---

## 6. Code Documentation

**Question**: Does this code documentation help the next developer (or future you) work with this code effectively?

- **README files answer "how do I get started?"** A module or project README must cover: what this is, how to install/set up, how to run/use it, and where to find more. If a developer clones the repo and the README doesn't get them to a running state, it failed its primary job.
- **API documentation describes the contract, not the implementation.** Document what the function/endpoint does, what inputs it accepts, what outputs it returns, and what errors it can produce. Do not document how it works internally — that changes, and stale internal documentation is worse than none.
- **CHANGELOG entries are for consumers, not authors.** "Refactored auth module" tells a consumer nothing. "Fixed: OAuth token refresh now correctly handles expired refresh tokens" tells them whether to upgrade. Group by impact (Breaking, Added, Fixed, Deprecated), not by date or commit.
- **Inline documentation explains why, not what.** The code already says what it does. Comments that restate the code add noise. Comments that explain why a non-obvious approach was chosen, why a seemingly simpler alternative doesn't work, or what business rule drives a condition — those prevent future developers from "fixing" things that aren't broken.
- **Type signatures and function names are documentation.** A function named `processData` with parameters `a`, `b`, `c` requires documentation. A function named `calculateShippingCost` with parameters `weight_kg`, `destination_zone`, `is_express` documents itself. Prefer self-documenting code; add comments only when the code cannot explain itself.

---

## 7. Everyday Technical Writing

**Question**: Are my PR descriptions, commit messages, and inline communications helping others understand intent — or creating noise that will be ignored?

- **PR descriptions succeed when they prevent the reviewer from asking "why?"** The diff already shows what changed. The description's job is to provide what the diff cannot: the motivation, the alternatives considered, the risks, and where the reviewer should focus. A PR with no description is not "self-explanatory" — it forces the reviewer to reconstruct intent, which means they review syntax instead of design.
- **The judgment in PR descriptions is deciding what context the reviewer needs.** A one-line bug fix may need a paragraph explaining the root cause and why this fix is correct. A 500-line feature may need only a summary if it implements an already-approved design doc. Match the description to the reviewer's knowledge gap, not to the diff size.
- **Commit messages serve two audiences at different times.** Right now, they help reviewers understand the change sequence. In six months, they are the primary artifact `git blame` and `git log` deliver to someone debugging. A commit message that serves neither audience ("fix", "WIP", "address comments") has negative value — it occupies space where useful information should be.
- **The judgment in commit messages is deciding when one line is enough.** A commit that renames a variable needs only a subject line. A commit that changes retry behavior needs a body explaining why the old behavior was wrong and what the new behavior guarantees. The signal: if someone reading the subject would ask "but why?", add a body.
- **Linking to context is not bookkeeping — it is knowledge preservation.** Reference the issue/ticket number, the design doc, or the incident that motivated the change. The link costs seconds to add and saves hours when someone needs to understand the decision six months later. A commit with no context trail is a dead end in the investigation chain.

---

## 8. Visual Communication

**Question**: Would a diagram, table, or visual aid communicate this more effectively than text?

- **Use diagrams for relationships and flows.** System architecture (what connects to what), data flows (how a request traverses services), state machines (valid transitions), and dependency graphs are all cases where a diagram communicates in seconds what paragraphs of text cannot. If you find yourself writing "A calls B, which then calls C, which returns to B, which updates D..." — draw it.
- **Use tables for comparisons and structured data.** Comparing three approaches across five dimensions? A table is processed in 10 seconds; five paragraphs take 3 minutes. Any time you are comparing options with multiple attributes, default to a table.
- **Use code examples for behavior.** When explaining how an API works or what a data structure looks like, a concrete example is worth more than an abstract description. Show the input and the output — the reader can generalize.
- **Keep diagrams simple and purposeful.** A diagram that tries to show everything shows nothing. Each diagram should have one purpose: the architecture overview shows components and connections (not data schemas); the sequence diagram shows interaction order (not error handling). Create multiple focused diagrams rather than one comprehensive one.
- **Diagrams must be maintainable.** Prefer text-based diagram formats (Mermaid, PlantUML, ASCII) over image files when possible. An image diagram becomes stale because nobody can edit it. A text-based diagram lives next to the code and can be updated with the change. If an image is necessary, document where the source file lives.
- **Not everything needs a visual.** Simple, linear explanations work fine as text. If the relationship is sequential with no branching or parallelism, a list or paragraph is clearer than a forced diagram. The test: would a reader understand this faster with or without the visual?
