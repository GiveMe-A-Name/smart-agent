# AI Agent Pitfalls

These are failure patterns specific to how AI coding agents operate. They are distinct from general engineering mistakes because they stem from the nature of LLM reasoning.

## Assumption Forcing

When requirements are ambiguous, the natural tendency is to fill in gaps with plausible-sounding assumptions rather than asking for clarification. This produces code that compiles and looks reasonable but solves a different problem than intended. When requirements leave room for interpretation on functional behavior, ask — don't guess.

## Over-Abstracting on First Pass

The tendency to introduce abstractions, interfaces, and generic patterns on the first implementation — before any concrete usage exists. A single concrete implementation is almost always better than a speculative abstraction. Write the concrete version first; abstract only when the second or third caller reveals a genuine pattern.

## Scope Creep Through "While I'm Here" Improvements

When touching a file, the pull to "fix" adjacent issues — rename a variable, restructure a helper, add a missing type annotation. Each is individually harmless; in aggregate, they make the diff unreadable and the change unreviewable. Stay on task. Note improvements for later; don't bundle them.

## Confident Hallucination of APIs and Behaviors

Generating code that uses plausible-but-nonexistent APIs, incorrect function signatures, or outdated library interfaces. Always verify external API usage against actual documentation or source code — never rely solely on training-data memory.

## Losing Track of Shared State During Refactoring

When splitting or moving functions across modules, losing track of what state was being read or modified. This is among the most common failure modes in AI coding agent research. Before any refactor that moves code across boundaries, explicitly trace every piece of state the code touches.

## Silently Closing an Existing Capability Path

When a fix, restoration, or simplification makes a currently reachable code path unreachable — a version-specific branch, a fallback mechanism, a native runtime path — the capability loss is not visible in the diff. Before implementing any change that eliminates a working path, explicitly identify what capability is being removed and surface it to the user before writing code.

## Implementing Toward an Unverified Goal

A reviewer's statement that "the goal hasn't been achieved" contains two claims: that the current code doesn't do X, and that X is the correct goal. The first is verifiable from code. The second must trace to the user's confirmed intent. If X was never explicitly confirmed by the user, implementing further toward it deepens the wrong objective.
