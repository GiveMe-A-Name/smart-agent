---
name: writing-plans
description: Invoke before any implementation unless you are executing an already-committed plan with no remaining decisions. Covers all sizes — even a Tiny plan (prose note + file path + verification) prevents mid-implementation surprises. When uncertain whether a plan exists or is complete, invoke.
---

# Writing Plans

A plan that doesn't match the task size is waste. A tiny bug gets a prose note. A new subsystem gets full task decomposition. Read the situation first — then write the plan that fits.

## Trigger Logic

**Invocation default**: Skipping a plan doesn't save time — it converts unknown unknowns into mid-implementation surprises. Even a Tiny plan (a prose note with a file path and verification step) forces the situation assessment that prevents wasted work. Invoke this skill before writing any code unless you are executing a plan that already exists and has no remaining decisions.

Use this skill when:
- you are about to implement a change and cannot immediately state the files involved, what specifically changes, and how to verify it
- the work involves multiple steps, files, or domains where sequencing or task boundaries affect the outcome
- the work is a refactor, migration, or architecture change where mapping what touches what matters before moving anything

Do not use this skill when:
- you are executing an already-agreed plan where scope, targets, and sequencing are settled
- the work is still open-ended exploration, root-cause investigation, or brainstorming with no settled implementation target yet

If prerequisite understanding is still missing — codebase evidence too thin, or intent still unclear — build that first. Return to this skill once the situation can be assessed.

## Capability Boundary

This skill owns the full planning cycle: from reading a spec to a plan ready for implementation.

It does NOT own:
- Execution of the plan
- Spec creation or open-ended brainstorming
- Root cause investigation
- Engineering judgment about how each change should land — a plan names targets and sequence; reasoning about approach, tradeoffs, and fit with existing structure belongs to the execution phase, not the planning phase

## Invariants

```
NO PLAN WITHOUT SITUATION ASSESSMENT FIRST
```

You cannot start writing until you can state: *"This is a [size] [nature] task. The relevant code is [X]. The constraint is [Y]."*

If you can't name the files involved, you haven't assessed enough.

**Before exiting this skill, you MUST complete the Self-Check section at the end.**

## Situation Assessment

Before writing anything, read three dimensions:

**Size** — Use behavioral signals, not just time estimates:
- **Tiny**: change fits inside one function/method, no new interfaces, no new test design needed
- **Small**: crosses function boundaries but stays in one concept/domain, existing test patterns apply
- **Medium**: introduces a new interface or concept, affects multiple domains, requires new test design
- **Large**: requires design decisions upfront, affects system architecture or multiple components

**Nature** — What kind of work?
- **Bug fix**: something is broken — assess root cause before planning. A plan without a known root cause is a guess.
- **Feature**: new capability — understand what already exists and what patterns to follow
- **Refactor**: restructure without changing behavior — map what touches what before moving anything
- **Architecture**: design decisions involved — identify constraints and trade-offs first

**Current state** — What exists today?
- Which files are directly relevant?
- What patterns does the codebase already use that this should follow?
- What constraints cannot be broken (API contracts, performance, compatibility)?

## Plan Depth

**Tiny** — Prose only, no task structure:
- What's the problem / goal
- Exact file path and location of the change
- What to change (concrete, not "add validation")
- How to verify it worked (exact command + expected output)

**Small** — Light checklist:
- Goal in one sentence
- Files to touch (exact paths)
- Ordered steps — no TDD ceremony unless the change touches risky shared state
- Verification

**Medium** — Task structure:
- Goal + approach (2–3 sentences)
- File map: which files change, what each is responsible for
- Tasks with clear boundaries; each task produces something verifiable
- Test coverage where behavior could regress

**Large** — Full plan:
- Architecture overview and key decisions
- File structure with responsibility mapping
- Tasks with TDD steps, commit points, exact commands
- Scope check: if multiple independent subsystems, split into sub-plans

## Quality Principles (all tiers)

Regardless of size, a usable plan is concrete:
- **Exact file paths** — not "somewhere in src/", not "the auth module"
- **Concrete changes** — not "add error handling", but what the code should actually do
- **Specific verification** — exact command to run and what passing looks like
- **One thing per step** — if a step has an "and", it's two steps

## Review Loop

For medium/large plans only: dispatch a plan-document-reviewer subagent (see plan-document-reviewer-prompt.md). Skip for tiny/small — overhead exceeds value.

## Execution Handoff

After saving to `docs/superpowers/plans/YYYY-MM-DD-<name>.md`, offer to execute the plan. The plan settles scope and targets; execution still requires judgment about how each change lands.

## Failure Signals

**Over-planning** — stop if you catch yourself:
- Writing TDD ceremony for a one-function change
- Producing a plan document heavier than the work itself
- Running a review loop on a tiny/small task
- Spending more time planning than the task would take to just do

**Under-planning** — stop if you catch yourself:
- Starting medium/large work without a file map
- No concrete verification step for a complex change
- Planning a bug fix without knowing the root cause

**Skipping assessment** — stop if:
- You started writing before reading the relevant code
- You don't know which files are involved

## Examples

See `examples.md` in this directory for Tiny / Small / Medium plan examples.

## Self-Check Before Exiting

- [ ] Did I follow all invariants (situation assessment first, plan matches task size)?
- [ ] Did I catch any failure signals (over-planning, under-planning, skipping assessment)?
- [ ] Are file paths exact and changes concrete (not vague)?
- [ ] Am I exiting because the plan is genuinely complete, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
