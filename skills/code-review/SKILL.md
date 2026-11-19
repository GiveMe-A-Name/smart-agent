---
name: code-review
description: "Evaluate code changes by tracing their intent, behavior, affected contracts, and merge risk. Use when a diff, staged or local changes, or a PR exists and the task is to review, audit, approve, or judge those changes."
---

# Code Review

Understand the change from repository evidence before producing findings. Do not review without changed code to inspect, and do not implement fixes unless also asked to make changes.

## Understand Before Judging

Start from the behavior meant to change—not diff order or the largest hunk. For each independent behavior delta, locate its semantic center (the branch, state transition, contract, or boundary that decides it) and read the primary implementation plus enough surrounding code to see how it is reached. Do not judge from a description, file list, hunk, symbol name, or test alone.

Establish from repository evidence:

- **Intent and operating context:** Treat descriptions and comments as hypotheses; label intent as unknown or conflicting rather than inventing it. Ground environment assumptions in callers, config, and deployment evidence—do not assume distributed deployment, competing writers, or worst-case usage without evidence.
- **Affected boundaries:** When a change alters a public interface, API, schema, config, error behavior, side effect, ordering, identity, or persistent state, search for and inspect repository-visible callers and consumers. Note external or uninspected consumers as coverage limits.

## Judge Design and Necessity First

Evaluate responsibility placement, affected contracts, and added complexity before local polish; report design flaws that invalidate substantial work before lower-impact findings.

- **Placement:** Cite repository evidence (current owner, boundary, convention, or simpler supported mechanism) for design concerns. Preference alone is not a defect.
- **Necessity:** Judge new defenses, abstractions, dependencies, and state against actual use—not hypothetical future scenarios. Unnecessary complexity can warrant changes even when functionality and tests pass: state the unsupported premise, concrete maintenance burden, and simpler alternative. Preserve cheap checks with concrete benefits.

## Judge Tests by Value

Passing tests do not prove necessity or effectiveness. When tests change—or before requesting a missing test—judge whether the protected behavior matters to current usage, whether assertions would actually fail if that contract broke, and whether existing checks already cover it. Review a mechanism and its tests together: testing invented behavior does not justify the behavior.

Read `references/test-judgment.md` when tests are added or modified and their necessity, effectiveness, or overlap needs detailed assessment.

## Ground Findings and Verification

- Every finding must tie a changed decision to a concrete consequence and file/line or check: a reachable failure condition for behavioral defects, or the unsupported scope/redundancy and specific maintenance burden for unnecessary code or tests (do not invent runtime failures to justify cutting bloat).
- Omit ungrounded claims, optional style polish (unless requested or materially affecting defect risk), and unrelated pre-existing defects outside the diff unless directly exercised or broken by the change.
- Inspect repository checks for touched behavior when available. Report executed checks as passed or failed and unexecuted checks as not run; never imply verification that did not occur. State material coverage limits without treating uncertainty as proof of a defect.

## Output

Keep the review proportional to scope and merge risk, leading with the highest risk:

- **Findings:** Ordered by merge risk, or explicitly state none were found. Include file/line, what is wrong, triggering condition or maintenance burden, concrete consequence, and a fix direction when supported.
- **Verification and coverage:** Executed vs. unexecuted checks, brief test judgment when tests changed, and material coverage limits.
- **Verdict:** Include `Ready to merge`, `Ready with fixes`, or `Not ready` when an approval or merge decision is requested.

Read `templates/review-output.md` only when a full structured review report (including explicit Change Model and Design Assessment sections) is needed.
