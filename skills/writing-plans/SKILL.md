---
name: writing-plans
description: Use when implementation work is substantial enough that a written plan should come before coding - for example a scoped feature, refactor, migration, or cross-module change with clear requirements where file mapping, sequencing, task breakdown, or verification strategy materially affect the outcome. Trigger on requests like 'write an implementation plan', 'map the files first', or 'plan this before coding', and on 'break this into steps' only when the work is genuinely multi-step and non-trivial. Do not use for trivial edits, already-planned execution, root-cause debugging, or open-ended brainstorming.
---

# Writing Plans

A plan that doesn't match the task size is waste. A tiny bug gets a prose note. A new subsystem gets full task decomposition. Read the situation first — then write the plan that fits.

## Capability Boundary

This skill owns the full planning cycle: from reading a spec to a plan ready for implementation.

It does NOT own:
- Execution — see superpowers:executing-plans or superpowers:subagent-driven-development
- Spec creation / brainstorming — see superpowers:brainstorming
- Root cause investigation — see superpowers:systematic-debugging

## Invariant

```
NO PLAN WITHOUT SITUATION ASSESSMENT FIRST
```

You cannot start writing until you can state: *"This is a [size] [nature] task. The relevant code is [X]. The constraint is [Y]."*

If you can't name the files involved, you haven't assessed enough.

## Situation Assessment

Before writing anything, read three dimensions:

**Size** — Use behavioral signals, not just time estimates:
- **Tiny**: change fits inside one function/method, no new interfaces, no new test design needed
- **Small**: crosses function boundaries but stays in one concept/domain, existing test patterns apply
- **Medium**: introduces a new interface or concept, affects multiple domains, requires new test design
- **Large**: requires design decisions upfront, affects system architecture or multiple components

**Nature** — What kind of work?
- **Bug fix**: something is broken — assess root cause before planning (see superpowers:systematic-debugging). A plan without a known root cause is a guess.
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

After saving to `docs/superpowers/plans/YYYY-MM-DD-<name>.md`, offer:
1. **Subagent-Driven** (superpowers:subagent-driven-development) — recommended for medium/large
2. **Inline** (superpowers:executing-plans) — batch execution in this session

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
- You planned a bug fix before running superpowers:systematic-debugging

## Examples

See `examples.md` in this directory for Tiny / Small / Medium plan examples.
