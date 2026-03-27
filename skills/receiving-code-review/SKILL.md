---
name: receiving-code-review
description: Use when review feedback exists and the next step is deciding what to implement, challenge, or clarify before changing code. The feedback source does not matter. Do not use for requesting or performing a review, ordinary implementation with no review feedback, or applying feedback that has already been accepted without further evaluation.
---

# Receiving Code Review

Evaluate feedback technically before acting on it. Code review requires judgment, not compliance.

## Trigger Logic

**Invocation default**: Implementing review feedback without evaluation leads to blind compliance — applying suggestions that are technically wrong for this codebase, breaking existing behavior, or gold-plating things that were never needed. The cost of invoking this skill on feedback that turns out to be straightforward is a short verification pass. The cost of skipping it is a wrong implementation that passes review but breaks in production.

Use when review feedback exists and the task is deciding what to do with it before touching code. The source does not matter — user-pasted comments, PR threads, screenshots, inline annotations, or structured output from a reviewer sub-agent all qualify.

Do not use when:
- no review feedback exists yet — the task is to write code, not evaluate feedback
- feedback has already been fully evaluated and accepted, and the only remaining step is mechanical implementation with no judgment remaining

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

**Source calibration:** Verification is always required. The depth of skepticism scales with how much context the reviewer demonstrably has about this codebase:
- *Human partner* — implement after understanding; they carry architectural and intent context by default
- *Dispatched reviewer sub-agent* — has full codebase access but may miss intent, history, or reasons behind current implementation; verify before implementing
- *External reviewer* — may not know platform constraints, architectural decisions, or why current code exists; requires more explicit verification

A technically wrong suggestion is wrong regardless of source.

**Verification:** Before implementing any suggestion, check: 
1. Is it technically correct for this codebase? 
2. Does it break existing functionality? 
3. Is there a reason the current implementation exists — grounded in usage evidence, not theoretical justification? "This supports testability" or "this could be extensible" answers "could this be justified?", not "is it justified?" Usage evidence (actual callers, actual non-default invocations in production code) answers the latter. 
4. Does it work on all supported platforms and versions? 
5. Does the reviewer understand the full context? Run these checks only after understanding all items in the feedback — partial understanding of the set invalidates per-item verification. If you cannot verify something, say so explicitly and ask for direction rather than proceeding blind.

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
