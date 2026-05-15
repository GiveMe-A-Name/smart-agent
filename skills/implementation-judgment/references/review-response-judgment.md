# Review Response Judgment

When this skill is invoked after receiving code review — meaning feedback has been accepted and the question is now how the change should land — apply this additional judgment.

## Implement the Reviewer's Intent, Not Their Literal Suggestion

A reviewer may correctly identify a problem but suggest the wrong fix. Before applying accepted feedback, identify the reviewer's stated concern, the suggested code location if any, the current responsibility owner implied by the code, and whether the suggested location changes public behavior, dependency direction, or ownership. If any of those cannot be identified from code or user statements, surface the gap before writing code.

## Preserve Structure Under Accepted Feedback

Accepted review feedback can still damage structure if implemented as a literal patch. Before writing code:

1. **Check current design direction from evidence.** Name the existing responsibility owner, dependency direction, and public contract affected by the feedback. If the accepted change alters any of them, surface that shift before implementation.
2. **Check the suggested location against responsibility ownership.** The reviewer may be right about the *what* but wrong about the *where*. If the suggested location would move validation, recovery, state ownership, or domain logic into a layer that does not currently own it, do not implement there without surfacing the ownership change.
3. **Check downstream effects from callers and tests.** If the feedback changes return values, side effects, error types, test expectations, or compatibility behavior, treat it as a contract change and surface it before implementation.

## Don't Let Review Feedback Override Structural Signals

The fact that a change was suggested by a reviewer does not exempt it from the same structural checks applied to any other change. Stop before implementation if the accepted feedback introduces an abstraction without a second concrete caller, scatters one concern across layers, changes behavior under an existing name, or reverses dependency direction.

## Scope the Response Appropriately

Review feedback sometimes reveals a larger issue than what the reviewer commented on. When implementing accepted feedback:
- If the fix is truly local, make it local.
- If the feedback exposed a pattern problem, scope the improvement to the immediate vicinity — do not let it expand into a full redesign.
- If the feedback requires a design change that goes beyond the original change's scope, flag this and get explicit agreement before proceeding.

## The Unverified Goal Problem

A reviewer's statement that "the goal hasn't been achieved" contains two claims: that the current code doesn't do X, and that X is the correct goal. The first is verifiable from code. The second must trace to the user's confirmed intent, not only the reviewer's articulation. If X was never explicitly confirmed by the user, implementing further toward it deepens the wrong objective.
