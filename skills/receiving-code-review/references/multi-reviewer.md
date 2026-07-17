# Multiple Reviewers

Use this guide when feedback overlaps, conflicts, or contains higher-level decisions that may invalidate lower-level work.

## Triage by Effect on the Change

Evaluate feedback in this order:

1. Material correctness, security, data-loss, crash, and contract risks.
2. Questions about the change's intent, scope, or responsibility boundary.
3. Implementation risks that remain relevant under the chosen direction.
4. Advisory and preference-driven items.

Do not spend time applying local cleanup to code that an unresolved higher-level decision may replace.

## Overlap and Contradiction

Reviewer convergence increases confidence that a concern deserves inspection, but it does not establish its relevance, consequence, or remedy.

When suggestions conflict, identify the goal and risk each reviewer is optimizing for. Prefer a synthesis only when it protects the current change without adding disproportionate scope or complexity. Escalate when the underlying goals conflict and available intent evidence cannot resolve the choice.

Evaluate overlapping remedies independently. Several reviewers may identify the same real concern while proposing different or over-engineered solutions.

## Reporting

Group comments that share one underlying concern and disposition. Surface contradictions and dependencies; do not silently choose a reviewer or produce inconsistent implementations across the same change.
