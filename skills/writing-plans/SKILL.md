---
name: writing-plans
description: "Plan before implementing when the path is unclear. TRIGGER when: you cannot immediately state the files involved, what changes, and how to verify it. DO NOT TRIGGER when: executing an existing plan, or the change is trivially scoped to a single known file."
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

It DOES define the plan document's full structure — including the Execution Log zone that gets populated during execution. Defining the format at planning time is part of planning, even though the log itself is filled in later.

## Invariants

```
NO PLAN WITHOUT SITUATION ASSESSMENT FIRST
```

You cannot start writing until you can state: *"This is a [size] [nature] task. The relevant code is [X]. The constraint is [Y]. Done means: [exact verification command → expected output]."*

If you can't name the files involved or state how you'll verify completion, you haven't assessed enough.


## Completion Criteria

- [ ] The plan states task size, nature, current state, and decomposition strategy.
- [ ] Each task is a vertical slice that delivers verifiable value, leaves the codebase working, and is ordered risk-first.
- [ ] File paths are exact and planned changes are concrete rather than vague.
- [ ] For medium or large work, contingencies for the riskiest assumptions and plan revision triggers were identified.
- [ ] For cross-boundary work, interfaces and handoffs are explicit.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the plan is genuinely complete.

Did I ignore any failure signals such as over-planning, under-planning, or horizontal slicing because the plan already looks orderly?

Am I exiting because the plan is genuinely complete, or because the outline now looks structured enough?

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

**What does done look like?** Before decomposing, state the exact verification command and what passing looks like. This is the overall acceptance criterion — it anchors the entire decomposition. Each task also needs its own verification signal. A task with no independently testable output is a horizontal slice in disguise, even if the final task has a clean integration test.

**When the request is "restore old behavior / keep original logic"**: this phrase is ambiguous by construction. Before planning, check whether reachable capability paths exist in the current codebase that were not present at the referenced baseline. If they do, "restore old behavior" has multiple valid interpretations and cannot be planned as a single action — a mechanical code revert that silently closes a reachable capability path is a capability regression, not a restoration. Name the interpretations explicitly and get the user's choice before writing any plan. (The full disambiguation heuristic, including how to classify the interpretations, lives in `requirement-clarification`.)

**For "historical behavior + new capability" conflicts**: before writing a plan, output a behavior matrix — input conditions × old behavior × current behavior × candidate target behavior. If any cell is a "?" — a behavior the user hasn't confirmed — resolve those cells with the user before planning. The matrix makes the conflict visible; planning without it converts a requirement ambiguity into a code decision made silently.

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

**Large work** (architecture, design decisions, many components): Full decomposition — but detailed only for the near tasks. Identify dependencies, risks, and unknowns. Structure early tasks to resolve the biggest uncertainties. Leave later tasks at goal-level until the early work reveals the real landscape. If the work contains independent subsystems, split into separate plans. Place cross-cutting constraints (invariants that apply to every task) at the top of the plan document, before the task list — constraints buried mid-document are easy to miss during execution.

### Think About Estimation and Uncertainty

Estimation is not prediction — it is communication of uncertainty. Great engineers don't produce precise estimates; they produce honest ones with calibrated confidence.

**Confidence intervals over point estimates.** "It will take 3 days" is a guess dressed as a fact. "2-5 days, likely 3, depending on whether the existing API supports batch operations" communicates the same information plus the uncertainty and what drives it. The range tells stakeholders where the risk lives.

**Identify the unknowns that drive the range.** Most tasks have a "fast path" (everything works as expected) and a "slow path" (you discover a complication). Name the specific unknowns that separate the two:
- "Fast: 2 days if the payment API supports idempotency keys. Slow: 5 days if we need to build our own deduplication."
- "Fast: 1 day if the existing test harness covers this case. Slow: 3 days if we need to build new test infrastructure."

The unknowns are the valuable part of the estimate — they tell you what to investigate first (risk-first ordering).

**Known unknowns vs. unknown unknowns.** Known unknowns can be named and estimated: "I don't know if the API supports X, but I can find out in 2 hours." Unknown unknowns are the things you don't know you don't know — they appear as surprises mid-implementation. The wider your estimate range should be when:
- The domain is new to you
- The code you're changing is complex and poorly tested
- The feature touches multiple systems you don't control
- There is no precedent in the codebase for this type of change

**Revise as you learn.** An estimate made at the start of a task should be updated as you learn more. The first task in a plan often reveals information that changes the estimate for later tasks. This is not a failure of estimation — it is the estimation process working correctly. Communicate updated estimates early, not when the deadline has already passed.

**Don't estimate what you can measure.** If the question is "how long does the migration take?" and you can run it on a test dataset, measure it. If the question is "will this approach work?" and you can build a 30-minute spike, spike it. Estimation is for when measurement is too expensive — prefer measurement when it's cheap.

### Plan for What Could Go Wrong

Experienced engineers don't just plan the happy path — they think about what happens when their assumptions are wrong.

**Contingency planning ("if approach A fails").** For medium/large work, identify the 1-2 riskiest assumptions in your plan. For each, ask: "If this turns out to be wrong, what do I do?" You don't need a detailed backup plan — just a named alternative and a rough sense of the cost of switching. This prevents the panic-driven pivots that happen when Plan A hits a wall and there's no Plan B.

**Plan revision triggers.** A plan is not a contract — it's a hypothesis about how the work will go. Name the conditions under which the plan should be revised rather than pushed through:
- A task takes more than 2x its estimate — the remaining estimates are probably wrong too
- An assumption is disproven (the API doesn't support what you expected, the data shape is different)
- A new requirement or constraint emerges mid-implementation
- The first 2-3 tasks reveal the decomposition was wrong — the boundaries are in the wrong places

When a trigger fires, pause execution and revise the plan. Pushing through a plan that no longer matches reality is not discipline — it's stubbornness.

**Cross-boundary coordination.** When the plan touches code owned by others (different modules, different teams, shared interfaces), the plan must identify the coordination points:
- **Interface agreements before parallel work.** If the plan involves building two sides of an interface (API provider and consumer, library and caller), define the contract first. Building both sides against an unspoken assumption creates rework.
- **Explicit handoff points.** Where does one piece of work end and another begin? What does "done" look like for the piece you own? If the boundary is ambiguous, name it in the plan — ambiguous boundaries cause duplicate work or gaps.

## Quality Principles (all tiers)

Regardless of size, a usable plan is concrete:
- **Exact file paths** — not "somewhere in src/", not "the auth module"
- **Concrete changes** — not "add error handling", but what the code should actually do
- **Specific verification** — the actual executable command and expected output, not a prose description; "run the tests" is not a verification step, `pytest tests/foo.py::test_bar -v → PASS` is
- **One thing per step** — if a step has an "and", it's two steps
- **Each task is a vertical slice** — delivers verifiable value, not just a layer
- **Working state after each task** — the commit point rule applies at every level
- **Targets, not methods** — describe what file changes and what the resulting behavior is; if a step reads like pseudocode or implementation instructions, it belongs in execution, not the plan

## Review Loop

For medium/large plans only: dispatch a plan-document-reviewer subagent (see plan-document-reviewer-prompt.md). Skip for tiny/small — overhead exceeds value.

## Execution Handoff

After saving to `docs/superpowers/plans/YYYY-MM-DD-<name>.md`, offer to execute the plan. The plan settles scope and targets; execution still requires judgment about how each change lands.

During execution, maintain an `## Execution Log` section appended to the bottom of the plan document — see **Execution Log** below.

## Execution Log

The plan document serves a dual purpose: a spec for what to do, and a recoverable record of what was done. This makes interrupted work resumable across conversations — a new session reads the log and knows exactly where things stand.

During execution, append an `## Execution Log` section at the bottom of the plan document. **Never edit the plan tasks section during execution** — it stays static as the source of truth.

**Append to the log:**
- Task completions: `[YYYY-MM-DD] Task N: complete`
- Failed attempts: what was tried, why it failed, how the approach changed
- Plan revision triggers: which assumption broke, how future tasks were updated

**Omit from the log:**
- Code snippets
- Anything already captured in the plan section

**If a plan revision trigger fires:** update the not-yet-started tasks in the plan section and record the reason in the log. Do not rewrite completed or in-progress tasks.

## Failure Signals

**Over-planning** — stop if you catch yourself:
- Writing TDD ceremony for a one-function change
- Producing a plan document heavier than the work itself
- Running a review loop on a tiny/small task
- Spending more time planning than the task would take to just do
- Planning tasks 6+ in full detail when tasks 1-3 haven't been done yet
- Describing HOW to implement rather than WHAT changes and WHERE — if a task reads like a code walkthrough, it has crossed into execution territory

**Under-planning** — stop if you catch yourself:
- Starting medium/large work without a file map
- Per-task verification written as prose ("run the tests") rather than as an executable command with expected output
- No concrete verification step for a complex change
- Planning a bug fix without knowing the root cause
- No dependency analysis for work crossing multiple modules
- No contingency for the riskiest assumption in a medium/large plan
- No plan revision triggers — the plan assumes everything will go as expected
- Cross-boundary work with no agreed interfaces or handoff points

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

