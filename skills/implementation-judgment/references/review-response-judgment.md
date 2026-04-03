# Review Response Judgment

When this skill is invoked after receiving code review — meaning feedback has been accepted and the question is now how the change should land — apply this additional judgment.

## Implement the Reviewer's Intent, Not Their Literal Suggestion

A reviewer may correctly identify a problem but suggest the wrong fix. When the feedback is "this validation should be centralized," the reviewer's intent is eliminating scattered validation — not necessarily the specific refactoring they proposed. Find the implementation that addresses the reviewer's concern while preserving the design integrity of the original code.

## Preserve Design Integrity Under Accepted Feedback

Accepted review feedback can still damage structure if implemented carelessly. Before writing code:

1. **Does the accepted change fit within the original design direction?** If yes, implement it as a natural extension. If the accepted change requires altering the design direction, state this explicitly — the design intent shift should be a conscious decision, not a side effect of review response.
2. **Does the reviewer's suggested implementation location align with where this responsibility should live?** The reviewer may be right about the *what* (this needs to change) but wrong about the *where* (it belongs in a different layer than they suggested).
3. **Are there ripple effects the reviewer didn't anticipate?** A change that looks local in the diff may have contract implications for callers, test changes, or compatibility concerns. Surface these before implementing.

## Don't Let Review Feedback Override Structural Judgment

The fact that a change was suggested by a reviewer does not exempt it from the same structural judgment applied to any other change. All judgment dimensions still apply. Review feedback that introduces the wrong abstraction is still the wrong abstraction. Review feedback that scatters complexity is still scattered complexity.

## Scope the Response Appropriately

Review feedback sometimes reveals a larger issue than what the reviewer commented on. When implementing accepted feedback:
- If the fix is truly local, make it local.
- If the feedback exposed a pattern problem, scope the improvement to the immediate vicinity — do not let it expand into a full redesign.
- If the feedback requires a design change that goes beyond the original change's scope, flag this and get explicit agreement before proceeding.

## The Unverified Goal Problem

A reviewer's statement that "the goal hasn't been achieved" contains two claims: that the current code doesn't do X, and that X is the correct goal. The first is verifiable from code. The second must trace to the user's confirmed intent, not only the reviewer's articulation. If X was never explicitly confirmed by the user, implementing further toward it deepens the wrong objective.
