# Judgment Dimensions — Expanded Reference

Use this reference when a skill creation or review needs deeper diagnosis than the main `SKILL.md` provides. Do not open it for straightforward wording edits where the affected principle or constraint is already clear.

---

## 1. Capability vs. Procedure Character

**Question**: Does this material need a capability skill, or would an SOP, a note, or local documentation serve it better?

**Key signals**:
- Procedural tasks have fixed steps and low variation: the same sequence applies every time regardless of context. These are fine as SOPs. Forcing them into a capability-skill framework adds ceremony without improving judgment.
- Judgment tasks have high variation, context-dependent paths, or decisions that only make sense after reading the situation. These need capability skills.
- Repo-local setup instructions, team conventions, or one-project rules belong in local documentation unless they teach reusable agent judgment across contexts.

---

## 2. Principles vs. Constraints Placement

**Question**: Is each rule placed in the part that matches its job?

**Principles** guide positive behavior:
- what the agent should optimize for
- what causal model should shape judgment
- what tradeoff should be preferred in ambiguous cases

**Constraints** bound negative behavior:
- what must not happen
- what evidence is required before proceeding
- what observable condition should stop or revise the work

**Misplacement signals**:
- A principle says "be careful" or "ensure quality" instead of naming an action-shaping policy.
- A constraint describes a desired quality rather than an observable boundary, evidence requirement, red line, or stop condition.
- A support file contains the only reason a principle or constraint exists.

---

## 3. Lean vs. Hollow

**Question**: Is the main file calibrated between hollow and bloated?

**Hollow direction** — necessary logic was moved out:
- The frontmatter `description` must be enough for pre-load discovery.
- The main `SKILL.md` must contain the principles and constraints needed to execute the skill.
- Support folders carry expanded examples, lookup tables, templates, scripts, and detailed references.
- If the agent must open a support file before it can execute the skill safely, the main file is hollow.

**Ceiling direction** — the main file became too long or too dense:
- A long main file causes attention dilution: later execution may underweight important rules.
- A short but dense constraint section can still overload the agent when many independent constraints compete without a unifying principle.

**Structure guidance**:
- `references/`: read for expanded reasoning, review heuristics, or lookup material.
- `examples/`: compare good, bad, or boundary cases.
- `templates/`: copy or adapt as a scaffold.
- `scripts/`: execute for deterministic checks.

See `references/lean-skill-patterns.md` only when this Lean vs. Hollow dimension fires: the main `SKILL.md` is becoming long, repetitive, or dense; principle or constraint logic may be moving into support files; or you need to decide whether material belongs inline or in a support folder.

---

## 4. Integration Integrity

**Question**: Does each important idea have one clear home?

**Key signals**:
- The same rule appearing in multiple places signals the skill was built by accretion, not by judgment.
- Before adding a new section, identify whether the idea already belongs in `## Principles`, `## Constraints`, or a support file. Strengthen the existing home first.
- Stop and compress when: two passages teach the same judgment in slightly different words; an example repeats what a bullet already says; a paragraph exists only to defend a rule that is already clear.
- Decision order: merge into `## Principles` or `## Constraints` -> tighten wording -> replace repeated explanation with one rule plus one example -> move heavy support material to the correct support folder -> add a new section only if the idea cannot fit into the existing structure.
- Shorter is not the goal. One clear home per idea is the goal.

---

## 5. Constraint Signal Quality

**Question**: Are constraints written as verifiable conditions, and are principles written as action-shaping guidance?

**High-signal constraints**:
- Red lines with explicit "never" / "must not" plus the observable condition that triggers them.
- Evidence requirements that name what must already be seen: files opened, call sites traced, tests run, user statements made.
- Stop conditions that say when to revise, not just "reconsider if something feels wrong."

**High-signal principles**:
- State the direction of judgment, not a vague quality bar.
- Include causal rationale when the rule could be rationalized away in a new scenario.
- Let the agent derive the right action in a structurally similar case that does not match the surface example.

**Hollow forms**:
- Advisory prose: "the skill should be clear", "ensure quality", "be thorough".
- "You should X" statements that cannot be confirmed without subjective judgment.
- Constraints that describe a desired quality rather than an observable condition.

**Transformation test**:
- Hollow: "The skill must teach judgment well."
- Better principle: "Skills teach judgment; they do not encode orchestration [because fixed workflow steps freeze the agent's choices before it has seen the case]."
- Better constraint: "Do not leave advisory prose where an observable condition or explicit red line is needed."
