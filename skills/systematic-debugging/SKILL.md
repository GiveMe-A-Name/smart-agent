---
name: systematic-debugging
description: "Find root causes, not just fixes — systematic diagnosis before changing code. TRIGGER when: something is broken or behaving unexpectedly and the fix is not yet confirmed. DO NOT TRIGGER when: building a new feature or refactoring with no current failure."
---

# Systematic Debugging

Debugging is an inverse problem: you observe effects and must infer causes. This skill equips you with the mental models, judgment signals, and red lines to find root causes — not a fixed procedure. Assess the situation, choose the right tool, and adapt as evidence emerges.

## Trigger Logic

**Invocation default**: The cost of unnecessary invocation is minor overhead. The cost of skipping it is a fix that doesn't hold or hides the real problem. Invoke before changing any code in response to a failure or unexpected behavior.

Use when:
- something is broken or behaving unexpectedly and the fix is not yet known
- a bug was reported but the root cause has not been established
- proceeding without investigation would require guessing what to change

Do not use when:
- the work is a feature or refactor with no current failure

## Capability Boundary

This skill owns the full debugging cycle: from a visible failure to a single verified fix. It does NOT own architectural changes — 3+ fix failures that keep exposing new coupling is an escalation signal, not a scope expansion.

## Invariants

```
NO FIXES WITHOUT ROOT CAUSE FIRST
```

Root cause means you can state: *"The bug is at [specific location] because [mechanism], which produces [symptom]."*

**Prediction test**: Can you predict what will happen if you make a specific change? If yes, you have a root cause. If no, you have a hypothesis — keep investigating.

**One variable at a time.** Change one thing per experiment. If you change three things and it works, you don't know which fixed it — and the other two may have introduced new bugs.

## Completion Criteria

- [ ] Root cause was established with evidence, not just a hypothesis.
- [ ] The bug can be stated as: "The bug is at [location] because [mechanism], which produces [symptom]."
- [ ] The fix outcome could be predicted before running it.
- [ ] The fix was verified against the original failure.
- [ ] Related bugs following the same pattern were checked.
- [ ] Environment, configuration, and build issues were ruled out before focusing on code logic.
- [ ] If significant time passed without progress, escalation happened with context instead of continued spinning.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the bug is genuinely fixed.

Am I treating a symptom disappearance as proof of root-cause resolution?

Am I exiting because the bug is genuinely fixed, or because the latest run happened to look good enough?

---

## Failure Signals

Stop and reassess immediately when:

| Signal | What it means | What to do |
|---|---|---|
| Editing before restating the exact failure | Diagnosis is still implicit | State the failure in specific terms first |
| Picked the stack-trace file as owner without proof | Symptom site confused with owning layer | Trace the failure path to the behavior source |
| Skipped competing explanations | Jumped to conclusion | Check expectation, fixture, environment before blaming implementation |
| Multiple changes at once | Can't isolate what fixed it | Revert. One variable at a time. |
| Weakening assertions or increasing timeouts | Hiding the symptom, not finding the cause | Keep the assertion; find the cause |
| "It's probably X, let me try that" | Hypothesis without evidence | State the evidence first |
| Fix attempt 3+ without questioning architecture | Fix loop masking a design problem | Escalate |
| "I don't know why it works now, but it does" | Accidental fix, not understood | Find out why. If you didn't fix it deliberately, it will regress |
| Adapting a reference pattern without reading it fully | Copying without understanding | Read the working example completely first |
| Debugging code when the error points to tooling | Wrong layer — it's a build/config problem | Switch to build/dependency debugging approach |
| Assuming the config is what you think it is | Config/env assumption | Print/log the actual resolved configuration |
| Debugging alone for 1+ hour without progress | Diminishing returns | Escalate with what you know, what you tried, and what you think the next step is |

*See `references/cognitive-traps.md` for the underlying bias mechanisms and countermeasures behind these signals.*

---

## Reading the Situation

Reach for the mental model that fits the observable signal. After each action, ask: *"Did this shrink my search space?"* If not, change approach.

| What you're seeing | What it suggests | Reach for |
|---|---|---|
| Clear error message pointing to a specific location | May be a direct hit, but verify — could be symptom site, not owner | **Symptom site ≠ Owning layer** — trace backward before editing |
| "It used to work" / something changed recently | Likely a regression | `git log`, `git bisect` — **divide and conquer through time** |
| Fails in full suite, passes in isolation | Test pollution or ordering dependency | `scripts/find-polluter.sh`, bisection — **reproduction** |
| Intermittent / flaky | Uncontrolled variable: timing, state, ordering | **Reproduction** — find the variable, see `references/condition-based-waiting.md` |
| Multi-component system, unclear which layer | Need to isolate the broken layer | **Observation** — instrument boundaries, check data at each crossing |
| You've formed a theory but haven't tested it | Risk of confirmation bias | **Search problem** — design an experiment that could DISPROVE your theory |
| 2+ failed hypotheses, feeling stuck | Assumptions may be wrong | **Check the plug**, question assumptions, get a fresh view — explain the problem aloud |
| Each fix exposes a new failure elsewhere | Structural/design problem, not a single bug | **Architecture escalation** — surface to human partner |
| Error deep in a call stack, unclear origin | Need to trace backward to the original trigger | **Causal chain depth** + `references/root-cause-tracing.md` |
| Fix worked but you want it to stay fixed | Need to prevent the bug class, not just this instance | **Causal chain depth** + `references/defense-in-depth.md` |
| Build fails after dependency update | Version conflict or breaking change in dep | **Build debugging** — read changelog between versions, check lock file diff |
| "Works on my machine" but fails in CI/staging | Environment difference | **Config/environment debugging** — diff runtime versions, env vars, config sources |
| Code looks correct but behavior is wrong | Configuration, feature flag, or stale artifact | **Config debugging** — print actual resolved config, clean caches, check feature flags |
| Error message references internal tooling, not your code | Build tool, bundler, or compiler issue | **Build debugging** — read the FIRST error (not last), check toolchain versions |
| Same code behaves differently for different users | Feature flag, A/B test, permissions, or data-dependent | **Data/state debugging** — check user-specific state, flags, permissions |

---

## Mental Models

Five thinking tools. Each is a lens for seeing a bug differently. Reach for whichever fits the situation. See `references/mental-models.md` for full detail.

**Debugging Is a Search Problem** — every action should eliminate possibilities. Bisect through code, time (`git bisect`), components, or input. Ask: *"What's the fastest experiment that eliminates the most remaining possibilities?"*

**Observation Over Theory** — read the full error; print actual values; instrument at boundaries. Theory formed without data is usually wrong and causes anchoring.

**Symptom Site ≠ Owning Layer** — the file/line where the error surfaces is not necessarily where the fix belongs. First question: *"What put the system in a state where this line fails?"* Check expectation, fixture, environment, and implementation before accepting the stack-trace location.

**Causal Chain Depth** — apply 5 Whys. Fix at the surface and the bug class recurs; fix at the root and it cannot. Every time you think you've found the cause, ask one more "why."

**Reproduction as Understanding** — if you can't make the bug happen on demand, you don't understand it yet. Make it consistent, minimal, and fast.

---

## Escalation

Stop debugging alone and escalate when:
- 3+ fix attempts have failed — your mental model of the system is likely wrong
- Each fix exposes a new failure elsewhere — structural problem, not a single bug
- The bug is in someone else's domain
- You can't reproduce it after reasonable effort
- Blast radius is high — mitigate first (rollback, feature flag), continue root cause analysis after

When escalating: state what you know (symptom, evidence gathered, hypotheses tested), what you don't know, and what you think the next step is. "It's broken" without context is not escalation.

---

## Supporting Techniques

- `references/root-cause-tracing.md` — tracing backward through a call stack to the original trigger
- `references/defense-in-depth.md` — after finding root cause, adding validation at multiple layers so the bug class cannot recur
- `references/condition-based-waiting.md` — when the bug involves timing or async behavior
- `scripts/find-polluter.sh` — when a test fails only in a full suite run
- `references/specialized-domains.md` — domain-specific heuristics for distributed systems, performance, concurrency, data/state, build/tooling, and config/environment bugs
- `references/mental-models.md` — full detail for the five mental models
- `references/cognitive-traps.md` — the seven cognitive biases that make debugging hard, with countermeasures

## Examples

See `examples/symptom-vs-root-cause.md` — the most common place to stop too early.

See `examples/symptom-vs-owning-layer.md` — how a stack-trace line can be the symptom site without being the owning layer.

See `examples/fixture-bypass-detection.md` — how bypass information can appear acknowledged while being set aside.
