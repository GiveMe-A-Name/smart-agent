---
name: diagnosing-visible-failures
description: Use when a specific test failure, CI error, stack trace, or other currently visible regression is already present and the agent needs to figure out what layer most likely owns the problem before editing code.
---

# Diagnosing Visible Failures

Turn a visible failure into grounded diagnosis before editing code.

For this skill, a concrete failure is currently visible through a failing test,
error output, stack trace, or clear runtime breakage in the present task
context, even if the agent cannot reproduce it from scratch in one command yet.

This skill is about diagnosis quality, not report shape. Its job is to stay
anchored to the visible failure, identify the likely owning layer, compare the
main competing explanations, and decide whether editing is grounded enough to
begin.

## Trigger Logic

Use this skill when:
- a concrete current failure is already visible
- the agent is about to edit code
- the missing step is diagnosing what likely owns the failure

Do not use this skill when:
- the main question is whether a reported bug still reproduces at all
- no concrete current failure is yet visible
- broad repository mapping is now the main deliverable
- the main task is writing tests, choosing test level, or shaping fixtures

## Boundary

This skill owns:
- pre-edit diagnosis for a concrete current failure
- separating symptom location from the layer that likely owns the behavior
- checking whether the failure most likely belongs to implementation,
  expectation, fixture, or environment
- comparing nearby patterns before fix work begins
- deciding whether diagnosis is grounded enough to start editing

This skill does not own:
- issue freshness checks or minimal repro gating for reported bugs
- broad repository-slice analysis as the main deliverable
- writing or selecting the overall testing strategy
- authoring integration test fixtures as the main output
- judging whether work is complete

## Diagnosis Heuristics

- Start by restating the concrete failing behavior in specific terms.
- Separate the symptom site from the layer that most likely owns the behavior.
- Check the main competing explanations before blaming the implementation:
  expectation, fixture, environment, and implementation.
- Stay anchored to the visible failure path until that path is exhausted.
- If broader ownership analysis is required, say so explicitly instead of
  pretending local diagnosis is still enough.

## Enough Grounding To Edit

Editing is grounded enough to begin when:
- the current failure can be restated clearly
- there is a plausible owning layer backed by evidence rather than proximity
- the main competing explanations have been compared rather than skipped
- the remaining unknowns are explicit and do not make the edit blind

Do not start editing when the likely owner is still just the nearest file, when
the failure could still just as easily belong to expectation or fixture logic,
or when the unknowns are large enough that any edit would mostly be guesswork.
Use nearby patterns and code evidence to support the owning-layer judgment, but
treat proximity as support, not proof.

## Self-Correction Signals

Stop and revise when:

| Signal | What it means | What to do |
|---|---|---|
| You are editing before restating the exact failure | Diagnosis is still implicit | Restate the current failure first |
| You picked the stack-trace file as owner without proof | Symptom and ownership are mixed together | Re-trace the failure path |
| You assume the test is wrong or the code is wrong too early | Competing explanations were skipped | Check expectation, fixture, and environment |
| You are broadening into repository mapping before the visible failure path has been exhausted | You escalated too early | Stay with the anchored failure path first |
| You exhausted the visible failure path but still cannot identify the likely owner without broader repository mapping | Local diagnosis is no longer enough | Make the need for broader ownership analysis explicit |
| You propose a fix before naming a diagnosis | You are patching symptoms | Finish diagnosis first |
| You want to weaken assertions or increase timeouts to move on | You are hiding the symptom | Keep the assertion and find the cause |
| You still cannot explain ownership after tracing the visible path | Local diagnosis is insufficient | State what kind of missing grounding remains instead of bluffing through it |
| The diagnosis sounds like generic caution, broad repository notes, or early fix speculation | The skill did not change behavior enough | Restate which specific failure evidence grounds each claim |

See `examples/symptom-vs-owning-layer.md` for a contrast pair showing how a stack-trace line can be the symptom site without being the owning layer.

See `examples/fixture-bypass-detection.md` for a contrast pair showing how bypass information can appear to be acknowledged while still being set aside in favor of stack-trace evidence.
