---
name: receiving-code-review
description: "Invoke when review feedback exists — PR comments, inline annotations, or user-pasted suggestions — and the task is deciding what to do with it before touching code. Cost of unnecessary invocation: a short verification pass. Cost of missing: blind compliance with technically wrong suggestions, or structural damage from uncritically applied feedback."
---

# Receiving Code Review

Evaluate feedback as a principal engineer would: against design intent, technical evidence, and architectural direction — not by social compliance.

The coding agent is often the entity with the most complete context about why a change was made and what it is supposed to achieve. A reviewer sees the diff; the agent holds the implementation intent, the constraints that shaped the approach, and the tradeoffs that were consciously accepted. This asymmetry is the foundation of every evaluation decision. Collaboration does not mean compliance.

## Trigger Logic

**Invocation default**: Implementing review feedback without evaluation leads to blind compliance — applying suggestions that are technically wrong for this codebase, breaking existing behavior, undermining design intent, or gold-plating things that were never needed. The cost of invoking this skill on feedback that turns out to be straightforward is a short verification pass. The cost of skipping it is a wrong implementation that passes review but breaks in production or silently degrades the architecture.

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
- Decide how accepted changes should land structurally — evaluation ends at whether and what to change, not at how to structure the change once accepted

## Invariants

**Verify before implementing.** Never implement a suggestion without first checking it is technically sound for this codebase and aligned with the design intent.

**You hold authoritative context.** When you implemented the code being reviewed, you carry the implementation intent. Never accept a reviewer's characterization of what the code should achieve, or what goal it was built toward, without tracing it against the intent you understood when making the change. The reviewer may be right about a bug while wrong about the goal.

**No performative agreement.** Never say "You're absolutely right!", "Great point!", "Thanks for catching that!", or any expression of gratitude. Actions speak — fix it and show the change.

**Clarify all before implementing any.** If any item in multi-item feedback is unclear, stop and ask before touching anything. Items may be related; partial understanding leads to wrong implementation.


## Completion Criteria

- [ ] Each comment was evaluated with the design intent in mind, not in isolation.
- [ ] Comments that challenge correctness were distinguished from comments that challenge design.
- [ ] Decisions were supported by evidence, not just arguments.
- [ ] All invariants were followed: verify before implementing, hold authoritative context, no performative agreement, clarify all first.
- [ ] No reviewer's characterization of the goal was accepted without tracing it to your own implementation intent.
- [ ] If feedback came from multiple reviewers, contradictions were surfaced instead of being silently resolved, and severity was triaged before processing linearly.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the feedback was genuinely evaluated.

Did I ignore any failure signals because a convenient resolution path already existed?

Am I exiting because feedback is genuinely evaluated and addressed, or because the current responses look coherent enough?

Did I accept any reviewer's framing of what the code should do without verifying it against my own implementation intent?

## Decision Signals

These questions shape judgment for each feedback item. They are not sequential steps — the signal that matters depends on the comment.

**What kind of claim is this?**
- Bug / logic error / security → verify against the codebase; likely valid regardless of source
- Design / architecture suggestion → evaluate against design intent; a different approach is not automatically better
- Style / preference → author's call unless the team style guide mandates otherwise
- Over-engineering (generality, future-proofing, premature abstraction) → grep for actual usage before accepting; if nothing calls it, consider removal
- Reviewer confusion → improve code clarity regardless of whether the specific suggestion is right; confusion is a signal about the code, not only about the reviewer

**Does the reviewer understand the code they are commenting on?**
- Do they know the constraints that shaped this approach?
- Are they applying patterns from a different codebase that don't translate here?
- Are they seeing full change context, or only one diff?

**Reviewer's premise vs. reviewer's observation**: A comment like "this doesn't achieve X" contains two claims: (1) the current code doesn't do X, and (2) X was the correct goal. Verify claim 1 against the code. Verify claim 2 against *your* implementation intent — not against what the reviewer says the goal is. If X was never explicitly confirmed, treat it as an unverified goal before building toward it. This is critical when X would alter global strategy, close existing capability paths, or conflict with capabilities added since the last explicit agreement.

**Source calibration:**
- Human partner: carries architectural and intent context by default — implement after understanding
- Dispatched reviewer sub-agent: has codebase access but may miss intent and history — verify before implementing
- External reviewer: may not know platform constraints or why current code exists — requires more explicit verification

A technically wrong suggestion is wrong regardless of source. A reviewer with limited context may still be right about a bug while wrong about design direction.

**Does this serve the original design goals, or a different set of goals?** A suggestion can be technically valid and still wrong for this code if it sacrifices a tradeoff the design intentionally made. Before accepting: name what dimension this suggestion optimizes, and whether that dimension was more important than what it costs.

**Verify usage claims with evidence, not arguments.** "Supports testability," "cleaner," "more extensible" are arguments. Grep before accepting claims about what depends on something, what would benefit from an abstraction, or what actual usage patterns exist. If nothing calls the suggested abstraction, flag removal rather than implementing it.


## Decision Heuristics

For each comment, reach one of three decisions:

**Accept and implement** when: the suggestion is technically correct AND aligned with design intent; it identifies a genuine issue regardless of design (bugs, security); or it improves code clarity without changing behavior.

For accepted changes that are non-trivial — structural changes, responsibility movement, abstraction introduction, or behavioral modification — decide how the change should land structurally before writing code. The reviewer identified what to change; structural judgment decides how it should land. Skipping this step is how accepted review feedback introduces architectural damage. For purely mechanical fixes (typos, naming, simple bug fixes, formatting), proceed directly.

**Push back** when: a suggestion breaks existing functionality; violates YAGNI; is technically incorrect for this stack; conflicts with established design intent without proposing why the intent should change; undoes a deliberate tradeoff without offering a better one; has legacy or compatibility justification for the current approach; or conflicts with the human partner's prior architectural decisions.

Use technical evidence — reference tests, code, or platform constraints. Never push back with just "I prefer it this way." State the design intent, the tradeoff, and the evidence. When the disagreement is architectural, involve the human partner. Technical facts and data overrule opinions and personal preferences.

**Clarify** when: you don't understand what the reviewer is suggesting; the suggestion is ambiguous between multiple interpretations; you need more context to evaluate; or implementing it would require changing the design intent and you want to confirm that's the reviewer's actual position.

Ask a specific question. "Can you clarify?" is useless. "Are you suggesting we move validation into the controller layer, or that we add a separate validation middleware? The former changes our error-handling contract with callers." is useful.


## Handling Multi-Reviewer Feedback

When feedback comes from multiple reviewers, triage before processing linearly.

**Triage order:**
1. **Blocking / critical** — bugs, security, correctness. Start here.
2. **Design-level (Layer 0–1)** — evaluate before lower-layer fixes; acceptance may invalidate downstream feedback.
3. **Implementation (Layer 2–3)** — group related comments; multiple related issues may resolve with one structural fix.
4. **Style / preference** — batch for a final pass; author's call unless style guide mandates.

If design-level feedback requires significant rework, state that lower-layer feedback will be addressed in the reworked code. Don't fix typos in code that's about to be rewritten.

**Contradictions**: Surface them — do not silently pick one side. **Overlap**: Convergence signals the problem is real; evaluate each proposed fix independently.

See `references/multi-reviewer.md` for detailed handling of contradictions, overlapping feedback, and volume management.


## Response Language

**Correct feedback:** `"Fixed. [what changed]"` — or just fix it in code without comment.

**Pushback:** State the technical finding, reference the evidence, ask a specific question. Example: `"The current approach co-locates validation with the handler because callers depend on receiving domain-specific errors (see tests in X). Extracting to middleware would genericize the errors. Was that the intent, or were you aiming for something else?"`

**If you pushed back and were wrong:** `"You were right — I checked [X] and it does [Y]. Fixing."` No apology, no defense. State the correction and move on.

**If the reviewer misunderstood:** Don't say "you misunderstood." Instead: `"I think the disconnect is [X]. The code does [Y] because [Z]. I'll add a comment to make this clearer."` Fix the code clarity even when the suggestion is wrong — the confusion is real and will affect future readers.

**GitHub inline comments:** Reply in the comment thread, not as a top-level PR comment — `gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`


## Failure Signals

Stop and reorient if:
- You are about to write "you're right", "great point", "thanks", or any variant — delete it, state the fix instead
- You are implementing any item before understanding all items in the feedback
- You are implementing a suggestion without having verified it against both the codebase and the design intent
- You are accepting a design suggestion without being able to articulate how it serves the original design goals
- You accepted a reviewer's characterization of what the goal is, or what the code should achieve, without tracing it to your own implementation intent — the reviewer's premise about the goal may be wrong even when their observation about the current code is correct
- You cannot verify a suggestion but are proceeding anyway — state the limitation instead
- You are implementing feedback that contradicts the design intent without explicitly acknowledging the intent change and getting confirmation
- A suggestion is technically wrong but social pressure makes direct pushback feel impossible — state the technical finding anyway. If you genuinely cannot raise it in context, flag it to your human partner with "Strange things are afoot at the Circle K." This is a signal to escalate, not permission to defer.
- You are about to implement non-trivial structural changes from review feedback without first deciding how they should land structurally
- Two reviewers gave contradictory suggestions and you silently picked one without surfacing the conflict
- You are processing 20+ comments linearly instead of triaging by severity and layer first
- You are fixing style issues in code that design-level feedback may require rewriting

**When a failure signal fires, reorient on local context — not the whole review:** (1) re-read the design intent source — commit messages, PR description, prior conversation; (2) trace the specific claim against your own implementation intent; (3) grep for actual usage if the signal involves patterns, abstractions, or dependencies.
