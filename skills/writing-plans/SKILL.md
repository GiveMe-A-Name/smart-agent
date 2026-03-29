---
name: writing-plans
description: Invoke before any implementation unless you are executing an already-committed plan with no remaining decisions. Covers all sizes — even a Tiny plan (prose note + file path + verification) prevents mid-implementation surprises. When uncertain whether a plan exists or is complete, invoke.
---

# Writing Plans

A great engineer does not jump into code. They break the problem down, find the risks, order the work so each step delivers value, and keep the codebase working at every point. This skill teaches you to plan like that engineer — not just listing steps, but thinking strategically about decomposition, dependencies, and incremental delivery.

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

---

## How to Think About Planning

### Understand Before You Decompose

Before breaking anything down, you need to genuinely understand the situation — not fill in fields, but reason about what you're facing.

**How big is this?** Not in hours — in uncertainty and blast radius. A change inside one function with no interface impact is fundamentally different from a change that introduces a new concept across multiple modules. Rough guideline for calibration:
- **Tiny**: fits inside one function/method, no new interfaces, no new test design
- **Small**: crosses function boundaries but stays in one domain, existing patterns apply
- **Medium**: new interface or concept, multiple domains, needs new test design
- **Large**: design decisions upfront, architecture or multi-component impact

**What kind of work is this?** A bug fix without a known root cause is not ready for planning — it's still investigation. A feature needs awareness of existing patterns so you extend rather than reinvent. A refactor needs a map of what touches what before you move anything. An architecture change needs constraints and trade-offs surfaced, not just tasks listed. Let the nature of the work shape how you think, not which template you reach for.

**What exists today?** Which files are involved? What patterns does the codebase already use? What constraints are immovable? If you don't know this, your plan will collide with reality. Go read the code first.

### Think About Decomposition

This is where real engineering thinking matters. There is no single "correct method" — the right decomposition emerges from reasoning about your specific situation. But there are mental models that experienced engineers rely on. Internalize these, then apply your judgment.

**Each step should teach you something.** When you break work into pieces, every piece should either deliver value or resolve uncertainty. If a piece does neither — if it's "set up some structures nothing uses yet" — it's wasted motion. The moment you notice a task that produces nothing observable, ask why it exists as a separate step.

**Get something working end-to-end as early as possible.** A hardcoded, simplified, ugly version that actually runs tells you more than a perfectly architected set of pieces that have never been connected. This is why experienced engineers instinctively avoid layer-by-layer work ("first all the models, then all the endpoints, then all the UI") — each layer alone teaches you nothing about whether the system works. You only discover real problems when things connect, and you want that discovery to happen early, not at the end.

**Tackle what scares you first.** If there's a part you're uncertain about — an integration you haven't done, a constraint you're unsure you can meet, a design question with no obvious answer — that should come early. Not because "risk-first is a best practice" but because if that part fails, everything built on top of it is wasted. The comfortable, well-understood parts won't surprise you — they can wait.

**Think about what blocks what.** Some tasks can't start until others finish. Some are completely independent. When you notice this, let it shape your ordering naturally. Don't force sequential work that could be parallel, and don't plan parallel work that actually has hidden dependencies.

**Boundaries between tasks should be points where things work.** After finishing a task, you should be able to stop, run the tests, and everything passes. If a task leaves the codebase broken mid-flight, that's usually a signal you're cutting the work the wrong way — by technical layer instead of by behavior.

**When components need to talk to each other, define the contract first.** If the work crosses module or service boundaries, agreeing on the interface between them lets each side be built and tested independently. This is particularly powerful for large work — but only reach for it when the situation calls for it.

**Plan detail should match your certainty.** You know a lot about the immediate next step. You know less about step 5. You know almost nothing about step 10 — and whatever you think you know will likely shift after earlier steps. So put detail where it matters: the near work. For later tasks, a goal and rough scope is enough. Engineers who plan every step in full detail upfront aren't being thorough — they're creating fiction that feels like certainty.

**Granularity is judgment, not formula.** The right size for a task is: small enough to hold in your head while doing it, large enough that completing it produces a meaningful result. If a task feels too big to think about clearly, split it. If it's so small it's trivial, merge it. This is feel, developed through practice — the examples show what it looks like.

### Scale the Plan to the Work

The thinking above always applies. What changes is how much you write down.

**Tiny work** (one function, one file, no new concepts): Think it through in a few sentences. Name the file, the change, and how you'll verify it.

**Small work** (a few files, one domain, existing patterns): A short list — what files, what order, what to check. No ceremony.

**Medium work** (new concepts, multiple domains, new test patterns): Think through ordering, risks, and how pieces fit. Write tasks with clear boundaries. Identify what could go wrong and whether your ordering addresses that.

**Large work** (architecture, design decisions, many components): Full decomposition — but detailed only for the near tasks. Identify dependencies, risks, and unknowns. Structure early tasks to resolve the biggest uncertainties. Leave later tasks at goal-level until the early work reveals the real landscape. If the work contains independent subsystems, split into separate plans.

## Quality Principles (all tiers)

Regardless of size, a usable plan is concrete:
- **Exact file paths** — not "somewhere in src/", not "the auth module"
- **Concrete changes** — not "add error handling", but what the code should actually do
- **Specific verification** — exact command to run and what passing looks like
- **One thing per step** — if a step has an "and", it's two steps
- **Each task is a vertical slice** — delivers verifiable value, not just a layer
- **Working state after each task** — the commit point rule applies at every level

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
- Planning tasks 6+ in full detail when tasks 1-3 haven't been done yet

**Under-planning** — stop if you catch yourself:
- Starting medium/large work without a file map
- No concrete verification step for a complex change
- Planning a bug fix without knowing the root cause
- No dependency analysis for work crossing multiple modules

**Horizontal slicing** — stop if you catch yourself:
- All tasks in one layer (e.g., "create all models", "create all endpoints")
- No task delivers end-to-end verifiable behavior
- The first task you can actually test is the last one
- Tasks that leave the codebase in a non-working state

**Skipping assessment** — stop if:
- You started writing before reading the relevant code
- You don't know which files are involved
- You haven't stated the decomposition strategy

## Examples

See `examples.md` in this directory for Tiny / Small / Medium / Large plan examples showing decomposition strategies in action.

## Self-Check Before Exiting

- [ ] Did I complete the situation assessment (size, nature, current state)?
- [ ] Did I choose and state a decomposition strategy?
- [ ] Does every task leave the codebase in a working state (commit point rule)?
- [ ] Are tasks ordered risk-first (hardest/most uncertain first)?
- [ ] Is this a vertical slice plan (each task delivers verifiable value)?
- [ ] Are file paths exact and changes concrete (not vague)?
- [ ] Did I catch any failure signals (over-planning, under-planning, horizontal slicing)?
- [ ] Am I exiting because the plan is genuinely complete, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
