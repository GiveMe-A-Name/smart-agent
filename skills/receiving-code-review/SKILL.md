---
name: receiving-code-review
description: Use when the user already has code review feedback and needs to decide what to implement, what to challenge, or what to clarify before editing code. Trigger when they share PR comments or ask things like 'address this review', 'is this feedback correct?', or 'help me respond to review'. Do not use for requesting a review, performing a fresh review, ordinary implementation with no review feedback, implementing comments that have already been accepted without further evaluation, or checking changes after review feedback has already been applied.
---

# Receiving Code Review

Evaluate feedback technically before acting on it. Code review requires judgment, not compliance.

## Trigger Logic

Use when review feedback already exists and the task is deciding how to handle it before changing code.

Common trigger signals:
- The user pastes PR comments, review threads, screenshots, or a summary of reviewer feedback
- The user asks to address, apply, respond to, or work through review comments
- The user asks whether a review suggestion is correct, necessary, safe, or worth pushing back on
- The user has multi-item feedback and it is not yet clear which items are valid, related, or blocked on clarification
- The user says a reviewer asked for a refactor, abstraction, API change, test change, or "proper" implementation and wants help deciding what to do
- The feedback comes from someone who may lack full codebase context, or the feedback conflicts with local architecture, compatibility constraints, or existing behavior

## Capability Boundary

This skill owns the evaluation and response to review feedback. It produces a decision per feedback item: implement, push back, or clarify — with technical reasoning for each.

It does NOT:
- Request or dispatch code reviews
- Verify that implementation is correct after making changes
- Write the implementation itself

## Invariants

**Verify before implementing.** Never implement a suggestion without first checking it is technically sound for this codebase.

**No performative agreement.** Never say "You're absolutely right!", "Great point!", "Thanks for catching that!", or any expression of gratitude. Actions speak — fix it and show the change.

**Clarify all before implementing any.** If any item in multi-item feedback is unclear, stop and ask before touching anything. Items may be related; partial understanding leads to wrong implementation.

## Decision Signals

**Source calibration:** Verification is always required before implementing. The depth of skepticism scales with how much context the reviewer demonstrably has about this codebase. Human partner feedback carries more context by default — implement after understanding. External reviewer suggestions require more explicit verification — they may not know the current implementation's reasons, platform constraints, or architectural decisions. But verification applies to both: a technically wrong suggestion is wrong regardless of source.

**Verification:** Before implementing any suggestion, check: (1) Is it technically correct for this codebase? (2) Does it break existing functionality? (3) Is there a reason the current implementation exists? (4) Does it work on all supported platforms and versions? (5) Does the reviewer understand the full context? Run these checks only after understanding all items in the feedback — partial understanding of the set invalidates per-item verification. If you cannot verify something, say so explicitly and ask for direction rather than proceeding blind.

**YAGNI:** When a reviewer suggests "implementing properly" — adding a feature, endpoint, or abstraction — grep the codebase for actual usage first. If nothing calls it, the right move is flagging removal, not more complete implementation.

**Push-back:** Push back when a suggestion breaks existing functionality, violates YAGNI, is technically incorrect for this stack, has legacy or compatibility justification, or conflicts with the human partner's prior architectural decisions. Use technical evidence — reference tests, code, or platform constraints. Involve the human partner if the disagreement is architectural.

## Failure Signals

Stop and reorient if:
- You are about to write "you're right", "great point", "thanks", or any variant — delete it, state the fix instead
- You are implementing any item before understanding all items in the feedback
- You are implementing a suggestion without having verified it against the codebase
- You cannot verify a suggestion but are proceeding anyway — state the limitation instead
- A suggestion is technically wrong but social pressure makes direct pushback feel impossible — state the technical finding anyway. If you genuinely cannot raise it in context, flag it to your human partner with "Strange things are afoot at the Circle K." This is a signal to escalate, not permission to defer.

## Response Language

**Correct feedback:** `"Fixed. [what changed]"` — or just fix it in code without comment.

**Pushback:** State the technical finding, reference the evidence, ask a specific question.

**If you pushed back and were wrong:** `"You were right — I checked [X] and it does [Y]. Fixing."` No apology, no defense. State the correction and move on.

**GitHub inline comments:** Reply in the comment thread, not as a top-level PR comment — `gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`
