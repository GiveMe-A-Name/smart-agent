# Planning Methodology

Reference material for the writing-plans skill. Contains detailed guidance on how to think about planning — what to assess, how to decompose, how to estimate, and how to handle risk. The main SKILL.md contains the law (invariants, completion criteria, failure signals); this file contains the reasoning behind it.

---

## Understand Before You Decompose

Before breaking anything down, reason about what you're facing — not fill in fields.

**How big is this?** Not in hours — in uncertainty and blast radius. See the Task Sizing table in SKILL.md for calibration. A change inside one function with no interface impact is fundamentally different from a change that introduces a new concept across multiple modules.

**What kind of work is this?** A bug fix without a known root cause is not ready for planning — it's still investigation. A feature needs awareness of existing patterns so you extend rather than reinvent. A refactor needs a map of what touches what before you move anything. An architecture change needs constraints and trade-offs surfaced, not just tasks listed. Let the nature of the work shape how you think, not which template you reach for.

**What exists today?** Which files are involved? What patterns does the codebase already use? What constraints are immovable? If you don't know this, your plan will collide with reality. Go read the code first.

**What does done look like?** Before decomposing, state the exact verification command and what passing looks like. This is the overall acceptance criterion — it anchors the entire decomposition. Each task also needs its own verification signal. A task with no independently testable output is a horizontal slice in disguise, even if the final task has a clean integration test.

---

## How to Think About Decomposition

There is no single "correct method" — the right decomposition emerges from reasoning about your specific situation.

**Each step should teach you something.** When you break work into pieces, every piece should either deliver value or resolve uncertainty. If a piece does neither — if it's "set up some structures nothing uses yet" — it's wasted motion.

**Get something working end-to-end as early as possible.** A hardcoded, simplified, ugly version that actually runs tells you more than a perfectly architected set of pieces that have never been connected. This is why experienced engineers instinctively avoid layer-by-layer work ("first all the models, then all the endpoints") — each layer alone teaches you nothing about whether the system works. You only discover real problems when things connect, and you want that discovery to happen early.

**Tackle what scares you first.** If there's a part you're uncertain about — an integration you haven't done, a constraint you're unsure you can meet — that should come early. If that part fails, everything built on top of it is wasted. The comfortable, well-understood parts can wait.

**Think about what blocks what.** Some tasks can't start until others finish. Some are completely independent. Don't force sequential work that could be parallel, and don't plan parallel work that actually has hidden dependencies.

**Boundaries between tasks should be points where things work.** After finishing a task, you should be able to stop, run the tests, and everything passes. If a task leaves the codebase broken mid-flight, you're cutting the work by technical layer instead of by behavior.

**When components need to talk to each other, define the contract first.** If the work crosses module or service boundaries, agreeing on the interface between them lets each side be built and tested independently.

**Plan detail should match your certainty.** You know a lot about the immediate next step. You know less about step 5. You know almost nothing about step 10 — and whatever you think you know will shift after earlier steps. Put detail where it matters: the near work. For later tasks, a goal and rough scope is enough.

**Granularity is judgment, not formula.** The right size for a task is: small enough to hold in your head while doing it, large enough that completing it produces a meaningful result. If a task feels too big to think about clearly, split it. If it's trivial, merge it.

---

## Scaling Plan Depth to Work Size

The thinking above always applies. What changes is how much you write down.

**Tiny work** — Think it through in a few sentences. Name the file, the change, and how you'll verify it.

**Small work** — A short list: what files, what order, what to check. No ceremony.

**Medium work** — Think through ordering, risks, and how pieces fit. Write tasks with clear boundaries. Identify what could go wrong and whether your ordering addresses that.

**Large work** — Full decomposition, but detailed only for the near tasks. Identify dependencies, risks, and unknowns. Structure early tasks to resolve the biggest uncertainties. Leave later tasks at goal-level until the early work reveals the real landscape. If the work contains independent subsystems, split into separate plans. Place cross-cutting constraints at the top of the plan document, before the task list — constraints buried mid-document are easy to miss during execution.

---

## Estimation and Uncertainty

Estimation is not prediction — it is communication of uncertainty.

**Confidence intervals over point estimates.** "It will take 3 days" is a guess dressed as a fact. "2-5 days, likely 3, depending on whether the existing API supports batch operations" communicates the same information plus the uncertainty and what drives it.

**Identify the unknowns that drive the range.** Most tasks have a "fast path" (everything works as expected) and a "slow path" (you discover a complication). Name the specific unknowns that separate the two:
- "Fast: 2 days if the payment API supports idempotency keys. Slow: 5 days if we need to build deduplication."
- "Fast: 1 day if the existing test harness covers this case. Slow: 3 days if we need new test infrastructure."

The unknowns are the valuable part — they tell you what to investigate first (risk-first ordering).

**Widen the range when:**
- The domain is new to you
- The code you're changing is complex and poorly tested
- The feature touches multiple systems you don't control
- There is no precedent in the codebase for this type of change

**Revise as you learn.** An estimate made at the start should be updated as you learn more. The first task in a plan often reveals information that changes the estimate for later tasks. Communicate updated estimates early, not when the deadline has already passed.

**Don't estimate what you can measure.** If you can run the migration on a test dataset, measure it. If you can build a 30-minute spike, spike it. Estimation is for when measurement is too expensive.

---

## Contingency Planning and Plan Revision

**Contingency planning.** For medium/large work, identify the 1-2 riskiest assumptions. For each: "If this turns out to be wrong, what do I do?" You don't need a detailed backup plan — just a named alternative and a rough sense of the cost of switching.

**Plan revision triggers.** A plan is a hypothesis, not a contract. Name the conditions under which it should be revised rather than pushed through:
- A task takes more than 2x its estimate — the remaining estimates are probably wrong too
- An assumption is disproven (the API doesn't support what you expected, the data shape is different)
- A new requirement or constraint emerges mid-implementation
- The first 2-3 tasks reveal the decomposition was wrong — the boundaries are in the wrong places

When a trigger fires, pause and revise. Pushing through a plan that no longer matches reality is not discipline — it's stubbornness.

**Cross-boundary coordination.** When the plan touches code owned by others (different modules, different teams, shared interfaces):
- **Define the contract first.** If building two sides of an interface, agree on it before building either side.
- **Explicit handoff points.** Where does one piece of work end and another begin? What does "done" look like for the piece you own? Ambiguous boundaries cause duplicate work or gaps.
