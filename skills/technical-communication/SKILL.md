---
name: technical-communication
description: "Improve reader-facing technical artifacts when the primary problem is communication quality — structure, clarity, evidence, audience fit, or operational usability. Covers design docs, RFCs, ADRs, postmortems, runbooks, READMEs, API docs, and PR descriptions. DO NOT TRIGGER for implementation planning: scope, files, sequencing, verification, or execution handoff."
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

- Before drafting body text, identify who will read the artifact and what action they need to take. If either is missing, stop [because every structural choice — detail level, ordering, vocabulary — depends on who reads and what they must do; drafting without this locks in the wrong choices before the first sentence].
- Decision, proposal, review, and change-summary artifacts must state the decision, recommendation, or change intent in the first paragraph [because most readers stop at paragraph 1; context buried below will not reach decision-makers]. Operable and reference artifacts must begin with an opening section that states when to use the artifact, required prerequisites, and the first check or command [because an operator working under pressure will act on the first visible instruction; if that is preamble, they act on the wrong thing].
- Any claim used to justify a decision, priority, risk, or action must include at least one supporting anchor: a number, a concrete condition, an example, a link, or observed evidence. If it has none, mark it as an assumption or revise it [because an unjustified claim cannot be challenged or verified, making the artifact untrustworthy].
- Any artifact expected to remain current as the system, interface, or operating procedure changes is a living artifact. Every living artifact (runbook, API docs, architecture overview, README) must name an update owner or source of truth before publishing [because a document without an update owner will become stale and mislead the next reader who trusts it].

## Decision Signals

Work through these dimensions for each artifact. The first two must be answered before writing begins. See `references/judgment-dimensions.md` for detailed signals in each dimension.

**Audience** — Who reads this and what action do they need to take?
- If the artifact's primary reader or required action is unclear, stop before drafting.
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

**Code Documentation** — Does this code documentation help the next developer work with this code?
- README: the test is whether a developer who clones the repo can reach a running state without asking anyone. If not, the README failed its primary job.
- API docs: document the contract (inputs, outputs, errors, behavior), not the implementation. Internal details change; the contract is what consumers depend on.
- Inline comments: explain why a non-obvious choice was made, not what the code does. A comment that restates the code adds noise; a comment that explains why a simpler alternative does not work prevents future "fixes."

**Everyday Technical Writing** — Does this PR description or commit message provide context the diff cannot?
- PR descriptions: the diff shows what changed; the description must supply motivation, risks, and where to focus review. A description that describes the diff is redundant.
- Commit messages: if a reader of the subject line would ask "but why?", add a body. "fix", "WIP", "address comments" have negative value — they occupy space where useful context belongs.
- Include a link to the issue, design doc, or incident that motivated the change; the link costs seconds and saves hours when someone traces the decision six months later.

See `references/judgment-dimensions.md` for expanded guidance on all dimensions including visual communication. See `examples/artifact-examples.md` for positive and negative examples.

## Failure Signals

Stop and reassess if:

- the artifact's primary reader or required action is unclear before drafting [every section will be miscalibrated; the artifact will be written for the author, not the reader]
- a decision/proposal/review artifact reaches paragraph 2 without stating the recommendation, change intent, or ask [most readers stop at paragraph 1; if the key point is not there, decision-makers will not reach it]
- an operable/reference artifact begins without an opening section stating when to use it, prerequisites, or the first check or command [an operator under pressure acts on the first visible instruction; preamble at the top means they act on the wrong thing]
- you are writing implementation detail for a leadership audience, or high-level summaries for the implementation team [mismatched detail ensures the artifact cannot be used: leadership cannot approve from implementation steps; engineers cannot implement from summaries]
- the artifact uses jargon the primary reader cannot be expected to know and does not define it on first use [unexplained jargon stops comprehension; a reader who does not understand a term stops trusting the document and starts skimming]
- a claim driving urgency, risk, or correctness has no number, example, condition, link, observed evidence, or explicit assumption label [an unsupported claim looks like a conclusion but cannot be evaluated; reviewers will either challenge everything or defer without real engagement]
- alternatives were not considered or not documented for a decision artifact [missing alternatives signal the author did not evaluate other options, forcing reviewers to raise alternatives instead of evaluating the proposal]
- the artifact is expected to stay current after publication but has no named update owner or source of truth [a document without an update owner will become stale; future readers will trust outdated content]
- a PR description or commit message lacks the context the diff cannot provide [the diff shows what changed, not why; without intent, reviewers check syntax rather than design, and future debuggers hit a dead end]
- code documentation fails its job: README cannot reach a running state, API docs describe implementation instead of contract, or an inline comment only restates the code [each failure means the documentation exists but does not do the work; misleading documentation is worse than none]

## Completion Criteria

- [ ] The artifact's structure, level of detail, and opening match its reader, required action, and vehicle.
- [ ] The opening follows the artifact type rule: recommendation/change intent in paragraph 1 for decision/proposal/review artifacts, or an opening section with entry conditions plus first check/command for operable/reference artifacts.
- [ ] Claims that justify urgency, risk, correctness, or priority include evidence or are explicitly labeled as assumptions.
- [ ] Each heading names a question, decision, or action a reader could search for; no heading is a generic label such as "Background" or "Overview" where a more specific heading is available.
- [ ] Update handling is explicit: immutable artifacts are superseded to change; living artifacts name an update owner or source of truth.

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is artifact-type: completion is verified by self-checking the produced artifact against the Completion Criteria checklist above. No separate user confirmation is required.

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the document is genuinely useful.

Am I confusing polished prose with reader usefulness?

Am I exiting because the document is genuinely clear and useful, or because it now reads smoothly enough to me?
