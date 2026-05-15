---
name: technical-communication
description: "Improve persistent technical artifacts so later readers can understand, decide, review, or operate without author context. TRIGGER when writing or revising design docs, RFCs, ADRs, postmortems, runbooks, READMEs, API docs, PR descriptions, or commit messages. DO NOT TRIGGER for implementation planning, isolated inline comments, or ephemeral messages."
---

# Technical Communication

Write durable technical communication artifacts that are read, understood, and acted upon. The value of a technical artifact is not in writing it — it is in the decisions it enables, the knowledge it preserves, and the alignment it creates when the author is not present to explain it.

## Trigger Logic

**Invocation default**: A durable technical artifact that nobody can use has negative value — it cost time to write and creates a false sense that the information has been communicated. The cost of applying writing discipline is a short review pass. The cost of skipping it is wasted effort, unread documents, and decisions that are relitigated because the artifact did not carry the needed context. Invoke when writing a persistent artifact that another person must read later to understand, decide, review, or operate without relying on you to fill in missing context.

Do not use when:
- writing only isolated inline code comments inside implementation
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

- Before drafting body text, identify who will read the artifact and what action they need to take from the user request, existing artifact, or a working note outside the final artifact. If either reader or action is missing, stop [because every structural choice — detail level, ordering, vocabulary — depends on who reads and what they must do; drafting without this locks in the wrong choices before the first sentence].
- Decision, proposal, review, and change-summary artifacts must state the decision, recommendation, or change intent in the first paragraph [because most readers stop at paragraph 1; context buried below will not reach decision-makers]. Operable and reference artifacts must begin with an opening section that states when to use the artifact, required prerequisites, and the first check or command [because an operator working under pressure will act on the first visible instruction; if that is preamble, they act on the wrong thing].
- Any claim used to justify a decision, priority, risk, or action must be backed by at least one supporting anchor available to the agent: a number, a concrete condition, an example, a link, observed evidence, or an explicit assumption. If no anchor is available, revise or remove the claim [because an unjustified claim cannot be challenged or verified, making the artifact untrustworthy].
- Any artifact expected to remain current as the system, interface, or operating procedure changes is a living artifact. Before publishing a living artifact, identify its update owner or source of truth from user instructions, repo conventions, existing document metadata, or a working note outside the final artifact [because a document without a maintenance path will become stale and mislead the next reader who trusts it].

## Decision Signals

Work through these dimensions as drafting controls for each artifact. See `references/judgment-dimensions.md` for detailed signals in each dimension.

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
- Claims that justify urgency, risk, or correctness need an evidence anchor available to the agent: a number, example, condition, link, observed evidence, or explicit assumption.

**Longevity** — Is the update expectation explicit?
- Point-in-time documents (ADRs, postmortems): supersede rather than update; do not append corrections.
- Living artifacts are expected to stay current after publication; the agent must identify who owns updates or where the maintained source of truth lives.

**Code Documentation** — Does this persistent code documentation help the next developer work with this code?
- README: the test is whether a developer who clones the repo can reach a running state without asking anyone. If not, the README failed its primary job.
- API docs: document the contract (inputs, outputs, errors, behavior), not the implementation. Internal details change; the contract is what consumers depend on.
- Inline comments only fall under this skill when they are part of a broader documentation artifact or documentation pass. In that case, they must explain why a non-obvious choice was made, not what the code does. A comment that restates the code adds noise; a comment that explains why a simpler alternative does not work prevents future "fixes."

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
- a claim driving urgency, risk, or correctness has no number, example, condition, link, observed evidence, or explicit assumption available from the artifact, user request, source material, conversation evidence, or a working note outside the final artifact [an unsupported claim looks like a conclusion but cannot be evaluated; reviewers will either challenge everything or defer without real engagement]
- alternatives were not considered or not documented for a decision artifact [missing alternatives signal the author did not evaluate other options, forcing reviewers to raise alternatives instead of evaluating the proposal]
- the artifact is expected to stay current after publication but no update owner or source of truth can be identified from user instructions, repo conventions, existing document metadata, or a working note outside the final artifact [a document without a maintenance path will become stale; future readers will trust outdated content]
- a PR description or commit message lacks the context the diff cannot provide [the diff shows what changed, not why; without intent, reviewers check syntax rather than design, and future debuggers hit a dead end]
- code documentation fails its job: README cannot reach a running state, API docs describe implementation instead of contract, or an inline comment inside an in-scope documentation pass only restates the code [each failure means the documentation exists but does not do the work; misleading documentation is worse than none]

## Completion Criteria

- [ ] The user request, source artifact, conversation evidence, or a working note outside the final artifact identifies the primary reader, the action that reader must take, and the document vehicle before body text is drafted or revised.
- [ ] The opening contains the required entry point for its vehicle: decision/proposal/review/change-summary artifacts state the recommendation, change intent, or ask in paragraph 1; operable/reference artifacts state when to use the artifact, prerequisites, and the first check or command in the first section.
- [ ] Every claim that justifies urgency, risk, correctness, or priority has at least one supporting anchor available from the artifact, user request, source material, conversation evidence, or a working note outside the final artifact.
- [ ] Every heading can be mapped to a reader question, decision, or action stated in the artifact or conversation evidence. Generic headings such as "Background" or "Overview" remain only when no more specific reader question, decision, or action is available in that evidence.
- [ ] Update handling has been resolved from user instructions, repo conventions, existing document metadata, or a working note outside the final artifact.

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is artifact-type: completion is verified by self-checking the produced artifact against the Completion Criteria checklist above. No separate user confirmation is required.

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to catch edits that improved prose without improving reader action.

Can I point to the user request, source artifact, conversation evidence, or a working note outside the final artifact for the reader, required action, vehicle, evidence anchors, and update handling?

Did any edit only make prose smoother without improving the entry point, evidence, structure, or operating usefulness?
