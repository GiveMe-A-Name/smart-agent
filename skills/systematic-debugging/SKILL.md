---
name: systematic-debugging
description: "Find root causes, not just fixes — systematic diagnosis before changing code. TRIGGER when: something is broken or behaving unexpectedly and the fix is not yet confirmed. DO NOT TRIGGER when: building a new feature or refactoring with no current failure."
---

# Systematic Debugging

Debugging is an inverse problem: you observe effects and must infer causes. This skill constrains an agent to make cause claims only from observable evidence, so the fix is intentional rather than a plausible edit that happens to quiet the symptom.

## Trigger Logic

**Invocation default**: The cost of unnecessary invocation is minor overhead. The cost of skipping it is a fix that doesn't hold or hides the real problem. Invoke before changing any code in response to a failure or unexpected behavior [because editing without a cause claim turns debugging into trial-and-error].

Use when:
- something is broken or behaving unexpectedly and no evidence-backed root-cause statement has been written
- a bug was reported but the causal mechanism has not been traced to a specific location or external condition
- proceeding would require choosing an edit before stating what observation the edit is expected to change

Do not use when:
- the work is a feature or refactor with no current failure [because there is no observed symptom to trace backward from]

## Capability Boundary

This skill owns the full debugging cycle: from a visible failure to a verified fix and, optionally, adding defense-in-depth to prevent the bug class from recurring.

This skill does not own feature work, opportunistic refactors, or architecture redesign. If 3+ fix attempts fail or each fix exposes new coupling, stop expanding the patch and escalate with evidence [because repeated local fixes indicate the agent's system model is wrong or the problem boundary is larger than debugging].

## Invariants

```
NO FIXES WITHOUT ROOT CAUSE FIRST
```

Root cause means you can state: *"The bug is at [specific location or external condition] because [mechanism], which produces [symptom]. Evidence: [file/line, command output, log, trace, config value, or reproduction result]."*

**Prediction test**: Before editing, write the expected observable result of the change. If the expected result cannot name a command, log, test, response, config value, or state transition that should change, treat the cause claim as a hypothesis and keep investigating [because a cause that predicts no observation cannot distinguish deliberate fix from coincidence].

**One variable at a time.** Change one cause variable per experiment. If multiple files must change for one cause, state the single causal claim that requires them. If unrelated variables change together and the run passes, revert or isolate the changes [because the agent cannot identify which edit fixed the symptom and which edit introduced risk].

**Failure statement first.** Before opening an edit, record the exact observed failure: command or user action, expected behavior, actual behavior, and the first concrete error/output/state mismatch [because unstated failures let the agent silently substitute a more convenient problem].

**Competing explanations must be ruled in or out from evidence.** For the observed failure, check only plausible candidates among expectation, fixture/test data, implementation, environment/config/build, dependency/tooling, and external service state. Do not blame the nearest code until the evidence names why it owns the behavior [because stack traces often identify the symptom site, not the owning layer].

## Completion Criteria

- [ ] The response or notes contain the exact observed failure: command/user action, expected behavior, actual behavior, and first concrete error/output/state mismatch.
- [ ] The root-cause statement names a specific location or external condition, the mechanism, the produced symptom, and the evidence used to support it.
- [ ] Before the fix, the expected observable result was stated as a command, log, test, response, config value, or state transition.
- [ ] The verification reran the original failing path or an explicitly equivalent reproduction and recorded the observed result.
- [ ] Plausible competing explanations were checked or ruled out with evidence; implausible categories were not padded into a fake checklist.
- [ ] Related bugs following the same mechanism were searched for or the reason no search was applicable was stated.
- [ ] If the failure showed any config/environment/build signal from Reading the Situation (e.g., "works on my machine", error references tooling, code looks correct but behavior wrong), the actual resolved environment/config/build input was observed or the reason it could not be observed was stated.
- [ ] If 1+ hour passed without progress, escalation included: observed failure, evidence gathered, hypotheses tested, current blockers, and next proposed observation.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to detect when the agent is substituting a plausible story for evidence.

Stop if the only proof is that the symptom disappeared once; add or rerun an observation that exercises the stated mechanism.

Stop if the explanation would still sound valid after a different edit; rewrite the cause claim until it predicts why this edit, not any edit, changes the observed behavior.

---

## Failure Signals

Stop and reassess immediately when:

| Signal | What it means | What to do |
|---|---|---|
| Editing before restating the exact failure | Diagnosis is still implicit, so the agent can solve a substituted problem | State command/action, expected behavior, actual behavior, and first concrete mismatch before editing |
| Picked the stack-trace file as owner without proof | Symptom site confused with owning layer; the stack trace may show where invalid state crashed, not where it originated | Trace the failure path to the behavior source |
| Skipped plausible competing explanations | Jumped to conclusion; untested candidates can explain the same symptom and make the edit arbitrary | Check expectation, fixture/test data, implementation, and any plausible environment/config/build/dependency explanations before blaming implementation |
| Multiple unrelated changes at once | The passing run cannot identify which edit fixed the symptom and which added risk | Revert or split until each experiment tests one causal claim |
| Weakening assertions or increasing timeouts | Hiding the symptom, not finding the cause; the broken mechanism remains executable | Keep the assertion or timing contract; find the mechanism that violates it |
| "It's probably X, let me try that" | Hypothesis without evidence; trial edits anchor the agent on the first plausible story | State the observation that supports X or design an experiment that can disprove it |
| Fix attempt 3+ without questioning architecture | Fix loop masking a design problem; repeated misses show the current system model is unreliable | Escalate with observed failure, tested hypotheses, and remaining uncertainty |
| "I don't know why it works now, but it does" | Accidental fix, not understood; the bug can regress when the accidental condition changes | Identify which mechanism changed before exiting |
| Adapting a reference pattern without reading it fully | Copying without understanding; omitted context may be the part that makes the pattern safe | Read the complete working example and list the relevant differences before adapting |
| Debugging code when the error points to tooling | Wrong layer; application edits cannot fix a build/config/dependency failure | Switch to build/dependency debugging approach and inspect the first tooling error plus resolved versions/config |
| Assuming the config is what you think it is | Config/env assumption; runtime inputs may differ from source files or memory | Print/log the actual resolved configuration, env var, flag, or tool version |
| Debugging alone for 1+ hour without progress | Diminishing returns; continuing repeats the same faulty search strategy | Escalate with observed failure, evidence gathered, hypotheses tested, blockers, and next observation |

*See `references/cognitive-traps.md` for the underlying bias mechanisms and countermeasures behind these signals.*

---

## Reading the Situation

Select the mental model from the observable signal. Treat each row below as an entry point, not a conclusion — the way a failure presents is a hypothesis about its class, not a confirmed classification. After each action, record what possibility was eliminated. If no possibility was eliminated, change approach.

| What you're seeing | What it suggests | Reach for |
|---|---|---|
| Clear error message pointing to a specific location | May be a direct hit, but verify — could be symptom site, not owner | **Symptom site ≠ Owning layer** — trace backward before editing |
| "It used to work" / something changed recently | Likely a regression | `git log`, `git bisect` — **divide and conquer through time** |
| Fails in full suite, passes in isolation | Test pollution or ordering dependency | The cause is in what the suite introduces that isolation does not (shared setup, global state, order-dependent mutation). Bisect to locate the source — **reproduction** |
| Intermittent / flaky | Uncontrolled variable: timing, state, ordering | Run multiple times first. Consistent failure → deterministic bug, not flaky — switch model. Variable failure → **Reproduction** — find the uncontrolled variable, see `references/condition-based-waiting.md` |
| Multi-component system, unclear which layer | Need to isolate the broken layer | **Observation** — instrument boundaries, check data at each crossing |
| You've formed a theory but haven't tested it | Risk of confirmation bias | **Search problem** — design an experiment that could DISPROVE your theory |
| 2+ failed hypotheses with no eliminated cause | Assumptions may be wrong | **Check the plug** — list the assumptions not yet observed directly, then verify the highest-impact one |
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

Five thinking tools. Select the one whose observable signal matches the current failure. See `references/mental-models.md` for full detail.

**Debugging Is a Search Problem** — every action must eliminate at least one named possibility. Bisect through code, time (`git bisect`), components, or input.

**Observation Over Theory** — read the full error; print actual values; instrument at boundaries. A theory formed before observation anchors the agent on the first plausible explanation.

**Symptom Site ≠ Owning Layer** — the file/line where the error surfaces is not necessarily where the fix belongs. First question: *"What put the system in a state where this line fails?"* Check expectation, fixture, implementation, and any plausible environment/config/build explanations before accepting the stack-trace location.

**Causal Chain Depth** — apply 5 Whys. Fix at the surface and the bug class recurs; fix at the root and it cannot. Every time you think you've found the cause, ask one more "why."

**Reproduction as Understanding** — if the bug cannot be triggered on demand, record which triggering condition is still unknown. Make the reproduction consistent, minimal, and fast.

---

## Escalation

Stop debugging alone and escalate when:
- 3+ fix attempts have failed — your mental model of the system is likely wrong
- Each fix exposes a new failure elsewhere — structural problem, not a single bug
- The bug is in someone else's domain
- You have attempted at least two distinct reproduction strategies and still cannot trigger the failure on demand
- Blast radius is high — mitigate first (rollback, feature flag), continue root cause analysis after

When escalating: state the observed symptom, evidence gathered, hypotheses tested, unresolved unknowns, and next proposed observation. "It's broken" without context is not escalation.

---

## Supporting Techniques

- `references/root-cause-tracing.md` — tracing backward through a call stack to the original trigger
- `references/defense-in-depth.md` — after finding root cause, adding validation at multiple layers so the bug class cannot recur
- `references/condition-based-waiting.md` — when the bug involves timing or async behavior
- `references/specialized-domains.md` — domain-specific heuristics for distributed systems, performance, concurrency, data/state, build/tooling, and config/environment bugs
- `references/mental-models.md` — full detail for the five mental models
- `references/cognitive-traps.md` — the seven cognitive biases that make debugging hard, with countermeasures

## Examples

See `examples/symptom-vs-root-cause.md` — the most common place to stop too early.

See `examples/symptom-vs-owning-layer.md` — how a stack-trace line can be the symptom site without being the owning layer.

See `examples/fixture-bypass-detection.md` — how bypass information can appear acknowledged while being set aside.
