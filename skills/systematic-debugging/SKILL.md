---
name: systematic-debugging
description: "Find root causes, not just fixes — systematic diagnosis before changing code. TRIGGER when: something is broken or behaving unexpectedly and the fix is not yet confirmed. DO NOT TRIGGER when: building a new feature or refactoring with no current failure."
---

# Systematic Debugging

Debugging is an inverse problem: observe effects, infer causes, and verify the inference. Use this skill when a current failure or unexpected behavior exists and the fix is not yet evidence-backed.

## Principles

- **State the observed failure before editing.** Record the command or user action, expected behavior, actual behavior, and first concrete error/output/state mismatch [because unstated failures let the agent silently substitute a more convenient problem].
- **Make root-cause claims from evidence.** A usable root cause has the shape: "The bug is at [specific location or external condition] because [mechanism], which produces [symptom]. Evidence: [file/line, command output, log, trace, config value, or reproduction result]." [because a fix chosen before this claim is trial-and-error].
- **Use prediction to separate cause from coincidence.** Before changing code, name the observable result that should change: command, log, test, response, config value, or state transition [because a cause claim that predicts no observation cannot distinguish an intentional fix from a lucky edit].
- **Test one causal variable per experiment.** If multiple files must change, tie them to one causal claim; if unrelated variables change together and the run passes, isolate them [because the agent cannot know which edit fixed the symptom and which edit added risk].
- **Rule competing explanations in or out from evidence.** Check only plausible candidates among expectation, fixture/test data, implementation, environment/config/build, dependency/tooling, and external service state [because stack traces often identify the symptom site, not the owning layer].
- **Trace ownership before fixing the visible crash site.** When the error surfaces deep in a call stack or boundary, ask what produced the invalid state at that point [because editing the symptom site can hide the failure while leaving the source executable].
- **Treat reproduction as understanding.** If the bug cannot be triggered on demand, record which triggering condition remains unknown and keep shrinking the reproduction [because a non-reproducible bug cannot be verified against the claimed mechanism].

## Constraints

- Do not use this skill for feature work, opportunistic refactors, or architecture redesign when no current failure exists.
- Before the first edit, the notes or response must contain the observed command/action, expected behavior, actual behavior, and first concrete mismatch.
- Before final response, the notes or response must contain the root-cause location or external condition, mechanism, produced symptom, and supporting evidence.
- Before final response, rerun the original failing path or an explicitly equivalent reproduction and record the observed result.
- Do not weaken assertions, loosen contracts, increase timeouts, or skip tests unless evidence shows the original expectation was wrong.
- Before editing the stack-trace file, nearest code, or implementation layer, record the evidence showing that layer owns the behavior.
- Do not debug application code when the first concrete error points to tooling, build, dependency, config, or environment; inspect the resolved tool/config/version input first.
- Do not adapt a reference pattern before reading the complete working example and listing the relevant differences.
- Stop expanding the patch after 3+ failed fix attempts, after each fix exposes a new failure elsewhere, or after 1+ hour without progress; escalate with observed failure, evidence gathered, hypotheses tested, unresolved unknowns, and next proposed observation.

## Reading the Situation

Select the mental model from the observable signal. Treat each row as an entry point, not a conclusion; after each action, record what possibility was eliminated. If no possibility was eliminated, change approach.

| Observable signal | What it suggests | Reach for |
|---|---|---|
| Clear error message pointing to a specific location | May be a direct hit, but could be symptom site | Trace backward before editing; see **Symptom Site != Owning Layer** |
| "It used to work" or something changed recently | Regression | Use `git log`, `git bisect`, changelog, config diff, or dependency diff |
| Fails in full suite, passes in isolation | Test pollution or order dependency | Bisect suite order and shared setup; find what full-suite execution introduces |
| Intermittent or flaky | Uncontrolled timing, state, or ordering variable | Run multiple times first; if variable, isolate the uncontrolled condition; see `references/condition-based-waiting.md` |
| Multi-component system with unclear broken layer | Need boundary isolation | Instrument boundaries and compare data at each crossing |
| Theory formed before a disconfirming test | Confirmation-bias risk | Design an experiment that could disprove the theory |
| 2+ failed hypotheses with no eliminated cause | Assumptions may be wrong | List assumptions not directly observed and verify the highest-impact one |
| Error deep in a call stack | Original trigger is upstream | Trace the causal chain backward; see `references/root-cause-tracing.md` |
| Primary fix passes but traced evidence shows the same invalid state can enter through another path | Bug class can recur | Add the smallest boundary check that rejects that invalid state, and record the bypass path it covers |
| Build fails after dependency update | Version conflict or breaking change | Read the first tooling error, changelog between versions, and lockfile diff |
| "Works on my machine" but fails in CI/staging | Environment difference | Diff runtime versions, env vars, config sources, flags, and generated artifacts |
| Code looks correct but behavior is wrong | Config, feature flag, stale artifact, or data state | Print actual resolved config/state, clean caches, and check flags |
| Same code behaves differently for different users | Data-dependent behavior | Check user-specific state, permissions, feature flags, A/B assignment, and input data |

## Mental Models

Select the model whose observable signal matches the failure. See `references/mental-models.md` for full detail.

- **Debugging Is a Search Problem.** Every action should eliminate at least one named possibility; bisect through code, time, components, or input.
- **Observation Over Theory.** Read the full error, print actual values, and instrument boundaries before forming a fix.
- **Symptom Site != Owning Layer.** The file/line where the error surfaces is not necessarily where the fix belongs; ask what put the system in a state where that line fails.
- **Causal Chain Depth.** Every time a cause seems found, ask one more "why" until the fix prevents the bug class rather than the surface symptom.
- **Reproduction as Understanding.** A failure that cannot be triggered on demand still has an unknown condition; make the reproduction consistent, minimal, and fast.

## Supporting Techniques

- `references/root-cause-tracing.md` — tracing backward through a call stack to the original trigger
- `references/condition-based-waiting.md` — when the bug involves timing or async behavior
- `references/specialized-domains.md` — domain-specific heuristics for distributed systems, performance, concurrency, data/state, build/tooling, and config/environment bugs
- `references/mental-models.md` — full detail for the five mental models and stuck-loop recovery signals

## Examples

See `examples/symptom-vs-root-cause.md` when the proposed fix addresses the visible symptom but not the mechanism.

See `examples/symptom-vs-owning-layer.md` when a stack-trace line may be the symptom site rather than the owning layer, including when bypass context changes what stack-trace evidence proves.
