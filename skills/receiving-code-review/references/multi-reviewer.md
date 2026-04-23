# Multi-Reviewer Feedback: Extended Handling Guide

## Contradictory Suggestions

Reviewer A says "extract this into a service." Reviewer B says "keep this co-located for simplicity." This is a design disagreement — not a vote to tally.

**How to handle:**
- Identify what each reviewer is optimizing for. Contradictions often arise from different implicit priorities: performance vs. readability, flexibility vs. simplicity, consistency with area X vs. area Y.
- Check whether both suggestions address the same concern from different angles, or different concerns entirely. If the latter, both may be valid and the apparent contradiction is an illusion.
- When the contradiction is genuine: state both positions clearly, explain which aligns with the design intent and why, and ask for resolution from the reviewers or escalate to the human partner.
- Never implement contradictory suggestions in different parts of the same change. Inconsistency within a single PR is worse than either option alone.

## Overlapping Feedback

Multiple reviewers flag the same issue, sometimes with different proposed fixes.

**How to handle:**
- Treat convergence as a signal the problem is real — but evaluate each proposed fix independently on its merits. The first reviewer's fix is not automatically better than the second's.
- Respond to all threads even if the fix is the same. Each reviewer should see their feedback was evaluated.

## Volume Management: Full Triage Protocol

When a PR has 20+ comments from multiple reviewers, linear processing is a failure mode — and a completion-faking risk. A response to every comment creates the appearance of complete handling, but if high-severity items were processed in order rather than by priority, design-level concerns were evaluated after implementation-level fixes they might have invalidated. Triage first.

**Prioritization order:**
1. **Blocking / critical** — bugs, security, correctness. These must be fixed regardless. Start here.
2. **Design-level (Layer 0–1)** — suggestions that question whether the change should exist or whether the approach is right. These may invalidate other feedback if accepted, so evaluate them before implementing lower-layer fixes.
3. **Implementation (Layer 2–3)** — correctness, error handling, test quality. Group related feedback — multiple comments about error handling may be addressed by a single structural fix rather than point-by-point patching.
4. **Style / preference** — author's call unless a style guide mandates otherwise. Batch these for a final pass.

If design-level feedback is accepted and requires significant rework, state explicitly that lower-layer feedback will be addressed in the reworked code rather than the current version. Don't fix typos in code that's about to be rewritten.
