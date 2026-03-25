---
name: systematic-debugging
description: Use when debugging a current bug, failing or flaky test, runtime error, regression, or stack trace before changing code.
---

# Systematic Debugging

Random fixes waste time and create new bugs. Every bug has a root cause — find it, then fix it once.

## Capability Boundary

This skill owns the full debugging cycle: from discovering a bug to implementing a single verified fix.

It does NOT:
- Claim the fix is complete without running verification
- Own architectural changes — 3+ fix failures that keep exposing new coupling is an escalation signal, not a scope expansion for this skill

## Invariant

```
NO FIXES WITHOUT ROOT CAUSE FIRST
```

Root cause means you can state: *"The bug is at [specific location] because [mechanism], which produces [symptom]."*

Naming a symptom is not a root cause. Naming a hypothesis is not a root cause. Root cause requires evidence.

## How to Investigate

**Read error messages completely.**
Don't stop at the first line. Identify: the error type, the full stack trace, the first frame in your own code (not library internals), and any data values in the message. "What the error says" and "where in your code it originated" are different questions — answer both before moving on.

**Reproduce before hypothesizing.**
If you cannot reproduce the failure consistently, you do not understand it yet. Run the failing thing in isolation. If it passes in isolation but fails in a full suite, use `find-polluter.sh`. Gather more data before theorizing.

**Check recent changes before theorizing.**
Most bugs are caused by recent changes. Before forming a hypothesis: `git log --oneline -20`, check recent dep or config changes. "What changed?" is faster and more reliable than "what could be wrong?"

**In multi-component systems, instrument at every boundary first.**
When failure could be in any layer (CI → build → deploy, API → service → database), do not guess which layer. Add logging at each component boundary, run once, and read where data breaks. One instrumented run costs less than three wrong hypotheses.

**Find a working example and read it completely.**
When a similar pattern exists in the codebase, find one that works and diff it against the broken case. List every difference between working and broken, however small. Do not assume a difference is irrelevant before testing it.

## Hypothesis Discipline

Form one hypothesis at a time, stated explicitly: *"I think X is the root cause because Y."*

Test with the smallest possible change — one variable at a time. If the hypothesis is wrong, form a new one from the evidence, not on top of the failed fix.

## Architecture Escalation

If 3 or more fix attempts each fail and each reveals a new coupling problem in a different place, stop. This pattern — each fix exposes a new symptom elsewhere — means the design is structurally wrong. Surface this to your human partner before attempting another fix.

## Failure Signals

Stop and return to investigation if you catch yourself:
- Proposing a fix before stating a root cause with evidence
- Saying "it's probably X, let me try that"
- Making multiple changes at once to see if something works
- Planning to verify manually instead of running the failing test
- Adapting a reference pattern without reading it fully first
- On fix attempt 3+ without questioning the architecture

**If your human partner says:**
- *"Is that not happening?"* — you assumed without verifying
- *"Will it show us...?"* — you skipped adding evidence-gathering
- *"Stop guessing"* — you proposed a fix without a stated root cause

## Supporting Techniques

- `root-cause-tracing.md` — tracing backward through a call stack to the original trigger
- `defense-in-depth.md` — after finding root cause, adding validation at multiple layers so the bug class cannot recur
- `condition-based-waiting.md` — when the bug involves timing or async behavior
- `find-polluter.sh` — when a test fails only in a full suite run

## Examples

See `examples/symptom-vs-root-cause.md` for annotated before/after showing how to distinguish a symptom from a root cause — the most common place to stop too early.
