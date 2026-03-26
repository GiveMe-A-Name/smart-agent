---
name: systematic-debugging
description: Use before changing any code when something is broken or behaving unexpectedly. Invoke by default — skipping structured debugging leads to symptom fixes that hide root causes.
---

# Systematic Debugging

Random fixes waste time and create new bugs. Every bug has a root cause — find it, then fix it once.

## Trigger Logic

**Invocation default**: Skipping structured debugging when something is broken leads to symptom fixes that mask root causes and create new failures. The cost of unnecessary debugging steps is minor overhead. The cost of missing them is a fix that doesn't hold — or hides the real problem. Invoke this skill before changing any code in response to a failure or unexpected behavior.

Use this skill when:
- something is broken or behaving unexpectedly and the fix is not yet known
- a bug was reported but the root cause has not been established
- proceeding without investigation would require guessing what to change

Do not use this skill when:
- the work is a feature or refactor with no current failure

**Entry point**: If a concrete failure is already visible, start with Ownership Diagnosis. If no visible failure exists yet, start with Investigation.

## Capability Boundary

This skill owns the full debugging cycle: from a visible failure or reported bug to a single verified fix.

It does NOT:
- Claim the fix is complete without running verification
- Own architectural changes — 3+ fix failures that keep exposing new coupling is an escalation signal, not a scope expansion for this skill

## Invariant

```
NO FIXES WITHOUT ROOT CAUSE FIRST
```

Root cause means you can state: *"The bug is at [specific location] because [mechanism], which produces [symptom]."*

Naming a symptom is not a root cause. Naming a hypothesis is not a root cause. Root cause requires evidence.

## Ownership Diagnosis

When a concrete failure is already visible, determine which layer owns it before hypothesizing or editing.

**Restate the failure in specific terms.** Not "the test fails" — but what value was expected, what was received, and at which assertion.

**Separate the symptom site from the owning layer.** The file or line where the error surfaces is where the symptom appears, not necessarily where the behavior should change. A stack trace points to the symptom site; ownership requires tracing the behavior back to its source.

**Check the four competing explanations before blaming the implementation:**
- **Expectation**: is the assertion itself wrong — testing for the wrong thing?
- **Fixture**: is the test setup producing bad input or state?
- **Environment**: is something outside the code (config, timing, dependencies) causing this?
- **Implementation**: is the production code actually wrong?

**Stay anchored to the visible failure path.** Do not broaden to repository mapping until the visible failure path is exhausted. If broader ownership analysis is required, name it explicitly rather than pretending local diagnosis is still enough.

## Enough Grounding to Edit

Editing is grounded enough to begin when:
- the current failure can be restated clearly
- there is a plausible owning layer backed by evidence rather than proximity
- the main competing explanations have been compared rather than skipped
- the remaining unknowns are explicit and do not make the edit blind

Do not start editing when the likely owner is still just the nearest file, when the failure could still belong to expectation or fixture logic, or when the unknowns are large enough that any edit would be guesswork.

## Investigation

**Read error messages completely.**
Don't stop at the first line. Identify: the error type, the full stack trace, the first frame in your own code (not library internals), and any data values in the message. "What the error says" and "where in your code it originated" are different questions — answer both before moving on.

**Reproduce before hypothesizing.**
If you cannot reproduce the failure consistently, you do not understand it yet. Run the failing thing in isolation. If it passes in isolation but fails in a full suite, use `find-polluter.sh`. Gather more data before theorizing.

**Check recent changes before theorizing.**
Most bugs are caused by recent changes. Before forming a hypothesis: `git log --oneline -20`, check recent dep or config changes. "What changed?" is faster and more reliable than "what could be wrong?"

**In multi-component systems, instrument at every boundary first.**
When failure could be in any layer, do not guess which layer. Add logging at each component boundary, run once, and read where data breaks. One instrumented run costs less than three wrong hypotheses.

**Find a working example and read it completely.**
When a similar pattern exists in the codebase, find one that works and diff it against the broken case. List every difference between working and broken, however small. Do not assume a difference is irrelevant before testing it.

## Hypothesis Discipline

Form one hypothesis at a time, stated explicitly: *"I think X is the root cause because Y."*

Test with the smallest possible change — one variable at a time. If the hypothesis is wrong, form a new one from the evidence, not on top of the failed fix.

## Architecture Escalation

If 3 or more fix attempts each fail and each reveals a new coupling problem in a different place, stop. This pattern — each fix exposes a new symptom elsewhere — means the design is structurally wrong. Surface this to your human partner before attempting another fix.

## Failure Signals

Stop and return to investigation if you catch yourself:

| Signal | What it means | What to do |
|---|---|---|
| Editing before restating the exact failure | Diagnosis is still implicit | Restate the failure in specific terms first |
| Picked the stack-trace file as owner without proof | Symptom site and owning layer are mixed | Re-trace the failure path to the behavior source |
| Assumed the test is wrong or the code is wrong without checking alternatives | Competing explanations were skipped | Check expectation, fixture, and environment before blaming implementation |
| Broadening to repository mapping before exhausting the visible failure path | Escalated too early | Stay anchored to the visible failure path first |
| Exhausted the visible failure path but still cannot identify the likely owner | Local diagnosis is insufficient | State explicitly what broader ownership analysis is needed |
| Proposing a fix before naming a diagnosis or root cause with evidence | Patching symptoms | Finish diagnosis and confirm root cause first |
| Weakening assertions or increasing timeouts to move on | Hiding the symptom | Keep the assertion; find the cause |
| Diagnosis sounds like generic caution or early speculation rather than evidence | The skill did not change behavior | Restate which specific failure evidence grounds each claim |
| Saying "it's probably X, let me try that" | Hypothesis without evidence | State the evidence that supports the hypothesis before testing |
| Making multiple changes at once | Can't isolate what fixed it | One variable at a time |
| Planning to verify manually instead of running the failing test | Verification is weak | Run the test |
| Adapting a reference pattern without reading it fully | Copying without understanding | Read the working example completely before using it |
| On fix attempt 3+ without questioning the architecture | Fix loop is masking a design problem | Escalate to architecture review |

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

See `examples/symptom-vs-owning-layer.md` for a contrast pair showing how a stack-trace line can be the symptom site without being the owning layer.

See `examples/fixture-bypass-detection.md` for a contrast pair showing how bypass information can appear to be acknowledged while still being set aside in favor of stack-trace evidence.
