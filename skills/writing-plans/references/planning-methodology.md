# Planning Methodology

Reference material for the writing-plans skill. Contains detailed guidance on how to think about planning — what to assess, how to decompose, how to estimate, and how to handle risk. The main SKILL.md contains the decision-critical principles and constraints; this file contains the reasoning behind them.

---

## Understand Before You Decompose

Before breaking work into tasks, produce an assessment from observable evidence. If any required item below is missing, continue assessment instead of writing the plan.

**Size evidence:** State the size using uncertainty and blast radius, not time. Evidence must name affected functions, modules, interfaces, services, or domains. A one-function change with no interface impact is different from a change that introduces a concept across modules.

**Work nature:** State whether the work is bug fix, feature, refactor, migration, or architecture change. Bug fixes require a named root cause before planning. Features require at least one existing pattern read or an explicit note that no precedent exists. Refactors require a caller/dependency map. Architecture changes require named constraints and trade-offs.

**Current state:** List the files read, the relevant behavior observed in each, existing patterns to copy, and constraints that cannot move. If the plan names a file or pattern not yet read, mark it as an assumption and verify before dependent tasks.

**Done state:** State the exact verification command and expected output before decomposing. This is the overall acceptance criterion. Each task also needs its own verification signal. A task with no independently testable output is a horizontal slice in disguise, even if the final task has a clean integration test.

---

## How to Think About Decomposition

There is no single "correct method" — the right decomposition emerges from reasoning about your specific situation.

**Each step must either deliver behavior or resolve named uncertainty.** For every task, write one sentence that names the behavior delivered or the uncertainty resolved. If the sentence only says "set up" or "create structure" without a verification signal, the task is a horizontal slice.

**Create an end-to-end behavior before expanding layers.** The first or second task should connect the smallest usable path through the system, even with hardcoded data, when the feature spans multiple components. If all early tasks create models, infrastructure, or schemas without a runnable behavior, the plan delays discovery until too late.

**Move critical uncertainty early.** Put a task near the front when it depends on an unverified integration, an API capability not yet observed, a performance/security constraint, or a user decision. If that task fails, downstream tasks built on the assumption would be wasted.

**Name dependency edges.** For each task, state what prior task or artifact it depends on. Mark tasks as parallel only when they do not modify the same files, do not depend on each other's output, and share a defined interface.

**Task boundaries must be working states.** Each task ends with a verification command or observable check. If a task intentionally leaves tests failing or a behavior path broken, split it or move the boundary.

**Define contracts before parallel cross-boundary work.** If the plan crosses module, service, or team boundaries, add an interface/contract task before work that depends on both sides. The contract must name inputs, outputs, errors, and ownership/handoff point.

**Match detail to known inputs.** Give step-by-step detail only for tasks whose required inputs are already known from assessment or earlier completed tasks. Later tasks whose steps depend on unresolved findings should be goal-level with a revision trigger.

**Split or merge by verification size.** Split a task if it has multiple independent verification commands, modifies unrelated behavior paths, or cannot be summarized by one purpose sentence. Merge tasks when each one has no independent behavior, uncertainty reduction, or verification signal.

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

---

## Stop Signals

A stop signal is a pre-declared condition under which the agent must pause and ask before continuing — not revise the plan autonomously, but surface the situation and wait for a decision. Stop signals are not failure conditions; they are expected forks where the right path requires human judgment.

Three categories cover most cases:

**Resource change** — the plan is about to introduce something that wasn't agreed to:
- "If resolving this requires a new external dependency, pause and confirm before introducing it."
- "If the fix requires changing the public API contract, pause before proceeding."

**Premise invalidated** — a core assumption the plan rests on turns out to be false:
- "If `on_complete` doesn't carry enough context to build a useful payload, pause before designing a workaround."
- "If the existing schema doesn't support this, pause — the plan assumed it did."

**Progress overrun** — a task takes significantly longer than expected, signaling the estimates for remaining work are wrong:
- "If any single task takes more than 2× its estimate, report current progress and remaining estimate before continuing."
- "If total elapsed time exceeds 4 hours, surface what's done and what's left."

Write stop signals as concrete conditions, not vague thresholds. "If something seems complex" is not a stop signal. "If adding retry logic requires a new external dependency" is.

---

## Explicit Exclusions and Conflict Priority

**Explicit exclusions.** State what the plan deliberately does not include. This is most valuable when the scope boundary is non-obvious — when a reasonable agent could plausibly infer that X is in scope, even though it isn't.

- "Not building a webhook management UI — URL is config-only."
- "Not handling partial failures — assumes all-or-nothing semantics."
- "Not adding authentication to the new endpoint in this iteration."

One or two lines is enough. The goal is to prevent execution-time scope expansion, not to enumerate everything that won't be built.

**Conflict priority.** When the plan has multiple goals that can pull in opposite directions, name the tiebreaker explicitly. An agent that hits a real conflict mid-execution will make the call itself if no priority is declared — and it will often get it wrong.

Format: `When [X] conflicts with [Y], prefer [X]. [One sentence of reasoning.]`

Examples:
- "When correctness conflicts with performance, prefer correctness. Performance can be improved post-launch."
- "When security conflicts with schedule, prefer security. Flag the blocker rather than working around it."
- "When completeness conflicts with code cleanliness, prefer completeness in this iteration — refactoring is a separate task."

Conflict priority belongs in the Human Review Section for medium/large work, not buried in a task.
