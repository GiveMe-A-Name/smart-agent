# Seven Elements: Positive and Negative Examples

Each element shows a passing and a failing pattern. The failure label names the specific failure mode, not a vague quality judgment.

---

## 1. Trigger Conditions

**Pass**
```
Use this skill when: the agent is about to write behavior-affecting code and has not yet assessed which layer owns the change [because skipping this assessment converts unknown unknowns into mid-implementation surprises that require rework].
Do not use when: the change is confirmed to be a rename with no logic change — observable from the diff, not inferred from the request.
```
Inline rationale on the trigger condition: agent can now judge new scenarios ("is this a doc-only change?") against the stated purpose, not just the literal trigger description.

**Fail** — _scenario enumeration, not state description_
```
Use this skill when:
- fixing a bug
- implementing a new feature
- responding to code review
```
Scenario lists miss every unimagined case. A new trigger state ("refactoring a test helper") is not covered and will be skipped. State-based triggers generalize; scenario lists do not.

**Fail** — _structurally correct trigger, but missing inline rationale_
```
Use this skill when: the agent is about to write behavior-affecting code and has not yet assessed which layer owns the change.
```
The trigger is state-based and will fire correctly for known cases. But without the rationale, an agent encountering a borderline case ("this is just a tiny one-line patch") cannot reason about whether skipping is safe. It will default to skipping. Add the why.

---

## 2. Boundary and Non-Goals

**Pass**
```
This skill owns:
- deciding where responsibility for a behavior should live
- evaluating whether a local patch deepens a structural concern

This skill does not own:
- root cause investigation (prerequisite — complete before invoking this skill) [because acting without code evidence produces structurally plausible but wrong implementations]
- final release judgment
```
Inline rationale on the non-obvious non-goal only: "root cause investigation is a prerequisite" is counterintuitive — an agent might assume investigation is part of this skill. The why ("acting without evidence produces wrong implementations") explains why this boundary exists and prevents the agent from rationalizing that it can start here. "Final release judgment" is self-evident — no rationale needed.

**Fail** — _routing disguised as boundary_
```
This skill does not own:
- root cause analysis → use systematic-debugging skill
- planning → use writing-plans skill
```
Test: delete named-skill references. What remains? Nothing — the boundary was routing instructions, not capability description. A real boundary survives deletion of all skill names.

---

## 3. Task Contract

**Pass**
```
## Invariants
- Never treat the task description as the full implementation truth [because task descriptions name what to change, not whether that is the right layer or abstraction].
- Never add an abstraction without a second concrete caller already present in the codebase [because a single caller cannot demonstrate that the abstraction boundary is in the right place].
- Never move complexity upward into a caller when it belongs in the module.
```
Inline rationale on the non-obvious invariants: the first two rules are counterintuitive enough that agents rationalize exceptions ("this task description is clear enough"). The why forecloses that rationalization. The third rule ("complexity belongs in the module") is directionally self-evident — no rationale needed.

**Fail** — _invariants present but no rationale on the non-obvious rules_
```
## Invariants
- Never treat the task description as the full implementation truth.
- Never add an abstraction without a second concrete caller already present in the codebase.
- Never move complexity upward into a caller when it belongs in the module.
```
An agent following "never treat the task description as truth" correctly for known cases will still rationalize an exception when the task description is unusually detailed. Without the why, it has no basis to resist that rationalization. Same for the abstraction rule. Add inline rationale to the non-obvious rules.

**Fail** — _function-type contract instead of mode-type_
```
Input: a code change request
Output: a reviewed and improved implementation
Process: read → assess → implement → verify
```
This describes a function call, not an operating mode. A skill runs across multiple turns with state; it is not a pipeline with a single input and output. Rewrite as invariants that govern all turns.

---

## 4. Completion Criteria

**Pass**
```
- [ ] Every task in the plan opens with one sentence explaining why it exists — checkable by reading the first line of each task block.
- [ ] The first task passes the cold-start test: read it in isolation without conversation context. If a new agent cannot identify the first file to open and the first concrete change to make, rewrite it [because a plan that requires conversation context to execute will silently fail when handed to a fresh agent].
```
Inline rationale on the non-obvious criterion only: the cold-start test requirement is counterintuitive — it is not obvious why a plan that "looks complete" might still fail. The why explains the failure mode. The first criterion (task has a why-sentence) is self-evident — no rationale needed.

**Fail** — _abstract quality judgment, not verifiable condition_
```
- [ ] The plan is clear and actionable.
- [ ] The implementation is well-scoped.
```
"Clear," "actionable," "well-scoped" require the agent to judge its own work quality — the judgment the skill is meant to constrain. These cannot be confirmed from evidence. Delete or rewrite as specific, checkable conditions.

**Fail** — _verifiable but tests format, not substance (proxy metric)_
```
- [ ] The skill has an Invariants section with at least three items.
- [ ] The skill has a Completion Criteria section with at least five checklist items.
- [ ] The skill has an examples folder.
```
All three are verifiable — count the items, check the folder. But a skill with three advisory-prose invariants, five vague checklist items, and two positive-only examples passes all three. The criteria measure structure, not whether the structure teaches anything. Test: could an agent satisfy every criterion here while producing a hollow skill? If yes, rewrite to test substance — "every invariant is verifiable without agent self-judgment" instead of "at least three invariants."

---

## 5. Verification Approach

**Pass**
```
## Verification Approach
This skill is artifact-type. Completion is verified by self-checking the output against the Completion Criteria checklist. No user confirmation required.
```

**Pass (dialogue-outcome-type)**
```
## Verification Approach
This skill is dialogue-outcome-type. Completion requires the user to explicitly confirm the chosen direction in conversation. A checklist that looks complete is not sufficient — the user must confirm.
```

**Fail** — _verification approach unspecified_
```
## Completion Criteria
- [ ] The design covers the core requirements.
- [ ] Edge cases have been considered.
```
No verification approach stated. Agent cannot distinguish "I believe I'm done" from "I am done." Does the agent self-check? Ask the user? The skill is silent. Add a Verification Approach section.

---

## 6. Examples

**Pass**
```
examples/
  positive-plan.md        — a complete medium plan that passes all criteria
  boundary-tiny-plan.md   — smallest valid plan; shows what to omit without becoming hollow
  negative-horizontal.md  — a plan that fails the vertical-slice criterion; annotated with failure mode
```
Three files, three purposes. The negative example names its failure mode — not just "this is bad."

**Fail** — _positive examples only_
```
examples/
  good-plan-1.md
  good-plan-2.md
```
Two positive examples align the agent to the center of the distribution. Without a negative or boundary example, the agent has no anchor for where the line is. Add at least one negative or boundary case.

---

## 7. Failure Modes / Anti-Patterns

**Pass**
```
Stop and revise when:
- you added a parameter to an existing abstraction instead of questioning whether the abstraction is still the right boundary [because adding parameters extends a broken boundary rather than fixing it — the compounding cost outlasts the deadline]
- you are implementing from the task description without having read the affected call sites [because call sites reveal implicit contracts the task description does not mention]
- you introduced an abstraction with only one concrete caller currently in the codebase
```
Inline rationale on the non-obvious signals: the first two are counterintuitive enough that agents rationalize exceptions under deadline pressure. The why forecloses that rationalization. The third signal ("one concrete caller") is directly derivable from the abstraction principle — no rationale needed.

**Fail** — _verifiable signals, but missing rationale on non-obvious ones_
```
Stop and revise when:
- you added a parameter to an existing abstraction instead of questioning whether the abstraction is still the right boundary
- you are implementing from the task description without having read the affected call sites
- you introduced an abstraction with only one concrete caller currently in the codebase
```
An agent that knows "stop when you add a parameter" will stop in the cases it recognizes. But under pressure ("this parameter is tiny, the abstraction is still basically right"), it will rationalize past the signal. Without the why ("adding parameters extends a broken boundary — the compounding cost outlasts the deadline"), the agent has no basis to resist. It will repeat the same failure in a slightly different form.

**Fail** — _advisory prose, not verifiable condition_
```
- Avoid over-engineering
- Make sure the implementation is clean and maintainable
- Be careful not to break existing behavior
```
None of these are checkable. "Over-engineering," "clean," "maintainable," "careful" require judgment without giving the agent a concrete stopping condition. Transform each into a verifiable state or delete it.

**Fail** — _completion-faking: signals look like self-correction but are themselves advisory prose_
```
Stop and revise when:
- you are about to add another section without reviewing the existing ones
- the skill does not feel complete
- you have not thought carefully about whether each element is strong enough
```
These appear to be failure signals but cannot be confirmed from evidence. "Does not feel complete" and "thought carefully" delegate the check back to agent self-awareness — the same problem the failure signals section is meant to prevent. A failure signal must name an observable state, not a mental posture. This is completion-faking: the section exists and looks like self-correction, but provides no actual stopping condition.
