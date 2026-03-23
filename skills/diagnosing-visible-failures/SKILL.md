---
name: diagnosing-visible-failures
description: Use when a specific test failure, visible CI error, stack trace, or other currently visible regression is already present and the agent needs to diagnose the likely owning layer before editing code.
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

## Constitution

- Do not edit code before you can restate the concrete failing behavior.
- Do not confuse the symptom site with the owning layer.
- Do not assume the implementation is wrong until you have considered
  expectation, fixture, and environment failure modes.
- Prefer code evidence and nearby patterns over first-fix intuition. Treat
  nearby patterns as support, not proof of ownership.
- Do not weaken assertions or widen tolerances just to remove the symptom.
- Do not spread edits across multiple layers while ownership is still unclear.
- Report unknowns explicitly instead of hiding them inside confident diagnosis.
- Produce a diagnosis and readiness-to-edit judgment, not a speculative fix.

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

## What Good Looks Like

A good result:
- starts from a concrete visible failure instead of a vague report
- distinguishes symptom site from owning layer
- considers the main competing explanations
- uses code evidence and nearby patterns to support the diagnosis without
  treating proximity as ownership proof
- names meaningful unknowns
- stops premature code changes when diagnosis is weak
- makes it clear whether fix work can begin responsibly

If the result sounds like generic caution, broad repository notes, or early fix
speculation, the skill did not change behavior enough.
