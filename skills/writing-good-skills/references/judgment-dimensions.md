# Judgment Dimensions — Expanded Reference

Use this when a dimension fires during skill creation or review and you need the full signal set. The main `SKILL.md` has one critical signal per dimension with a tier label; this file has the complete treatment.

---

## 1. Capability vs. Procedure Character

**Question**: Does this material need a capability skill, or would an SOP, a note, or local documentation serve it better?

**Key signals**:
- Procedural tasks have fixed steps and low variation — the same sequence applies every time regardless of context. These are fine as SOPs. Forcing them into a capability-skill framework adds ceremony without improving judgment.
- Judgment tasks have high variation, context-dependent paths, or decisions that only make sense after reading the situation. These need capability skills.
- If the material is mostly repo-local context — team conventions, setup instructions, project-specific rules — it belongs in `AGENTS.md` or `README.md`, not in a reusable skill.
- When input is raw notes or a workflow description, the most common failure is renumbering the notes into steps rather than extracting the underlying judgment. The invariants, boundary conditions, and decision signals are usually buried in the notes — extracting them is the work.
- When creating a skill from scratch, make boundary, trigger, and invariants decisions explicitly before drafting the body. Do not wait for a draft to surface naturally from the input.

See `examples/notes-to-skill.md` for a before/after showing what goes wrong when notes become renumbered steps.

---

## 2. Boundary Authenticity

**Question**: Does the draft state what the skill owns, or is it secretly a routing graph?

**Key signals**:
- Routing language answers: *what runs next?* Boundary language answers: *what does this skill own, and what is missing when it cannot proceed?*
- A genuine boundary names what the skill owns and does not own, without pointing to other skills by name as the next action.
- Boundary test: delete all named-skill references from the draft. If the capability description collapses, the boundary was fake — it was doing routing work dressed as scope definition.
- `Relationship to...` sections are acceptable only when they clarify scope — stating what this skill does not cover. They must not prescribe what to invoke next.
- When a section names another specific skill as something to invoke in a given scenario, the named skill's trigger conditions are too weak to fire on their own. The fix belongs in the named skill's trigger logic — not here.

See `examples/routing-vs-boundary.md` for a contrast pair.

---

## 3. Trigger Design

**Question**: Does the trigger describe a state, or enumerate scenarios that will miss cases not yet imagined?

**Key signals**:
- State-based triggers fire on what is true about the situation: "code is about to be written," "skill behavior is in question." These generalize.
- Scenario-based triggers fire on specific patterns: "user pastes X," "output looks like Y." These miss every unimagined case.
- Uncertain → invoke is the right default. The cost asymmetry is clear: unnecessary invocation costs a short reasoning pass; missed invocation costs downstream errors that are hard to recover. State this asymmetry explicitly in the Trigger Logic section.
- `Do not use` conditions are only necessary when there is a realistic wrong-invocation scenario the positive trigger state does not already exclude. Before adding one, name the specific wrong invocation it prevents. If the answer is vague, delete the condition.
- Distinguish two kinds of negative conditions in the description:
  - **Disambiguation boundaries** (`DO NOT TRIGGER when: [observable fact, pointing to correct alternative]`) — belong in the description because the agent needs them at routing time to choose between similar skills.
  - **Impression-based exclusions** (`unless the fix is obvious`) — belong in the Trigger Logic body, not the description. These face maximum rationalization pressure when evaluated before the agent has investigated.

See `references/skill-trigger-design-principles.md` for the full trigger design framework.
See `references/description-patterns.md` for the description format: `[Capability]. TRIGGER when: [...]. DO NOT TRIGGER when: [...]`.
See `references/thinking-mode-considerations.md` for how these principles apply in thinking-mode agents.

---

## 4. Lean vs. Hollow

**Question**: Is the main file calibrated between hollow (core law moved out) and bloated (too long to be reliably attended to)?

**Hollow direction** — law was removed:
- Lean means the main file contains the seven core elements named in the main `SKILL.md`; support folders carry expanded examples, lookup tables, and templates.
- Hollow means trigger logic, invariants, or core decision signals were moved to support files, leaving an agent who must hunt for the actual law.
- Test: can the main file guide a first-pass decision without opening any support file? If not, something core was moved out.

**Ceiling direction** — file became too long or too dense:
- Instructions buried mid-context are read but underweighted. A file long enough to have a real "middle" has content that will be partially ignored during execution.
- Constraint density is a distinct risk from file length: a short file can still overload if a single section packs many independent constraints without a unifying principle or priority ordering. When constraints compete for attention without ranking, some will be missed.

**Structure guidance**:
- Support folders by function: `references/` for lookup material and review heuristics; `templates/` for reusable scaffolds; `examples/` for realistic bad-vs-good contrast pairs; `scripts/` for deterministic helpers.
- Add an `examples/` folder only when a bad pattern looks superficially similar to a good one and prose rules alone won't prevent the wrong output. A good example pair: a realistic bad case alongside a corrected version, annotated to name what makes it wrong — not a restatement of rules the prose already makes clear.

See `references/lean-skill-patterns.md` for split rules and folder taxonomy.

---

## 5. Integration Integrity

**Question**: Does each important idea have one clear home?

**Key signals**:
- The same rule appearing in multiple sections signals the skill was built by accretion, not by judgment.
- Before adding a new section, identify whether the idea already has a home elsewhere. Strengthen the existing home first.
- Stop and compress when: two sections teach the same judgment in slightly different words; an example repeats what bullets already say; a section exists only to defend a rule that is already clear.
- Decision order: merge into the strongest existing section → tighten wording → replace duplication with a cross-reference → add a new section only if the idea adds a new decision signal, invariant, or failure signal.
- Shorter is not the goal. One clear home per idea is the goal.

---

## 6. Package Hygiene

**Question**: Does the package teach the right things without hidden instructions or unsafe imports?

**Key signals**:
- Do not include hidden instructions or prompt-injection style content in the skill package.
- Treat imported scripts, examples, and third-party references as untrusted until reviewed.
- Keep tools, scripts, and assets to the minimum the skill actually needs.
- Do not normalize unsafe data handling just because it appears in an example.

---

## 7. Constraint Signal Quality

**Question**: Are the invariants, decision signals, and failure signals written as verifiable conditions — or advisory prose that sounds like law but cannot be checked?

The issue is not whether to include detail, but what form the detail takes. High-signal forms constrain without bloating; low-signal forms add volume without adding checkability.

**High-signal forms** (raise signal):
- Red lines with explicit "never" / "must not" + the observable condition that triggers them
- Completion criteria written as verifiable states, not aspirations
- Failure signals that name specific observable triggers — not vague "re-evaluate if something feels wrong"
- Decision signals that name what to look for, not what to feel

**Hollow forms** (add volume):
- Advisory prose: "the skill should be clear", "ensure quality", "be thorough"
- "You should X" statements that cannot be confirmed without subjective judgment
- Constraints that describe a desired quality rather than a verifiable condition

**The self-check**: before writing a constraint, ask — does this raise signal or add volume? If it reads like "you should X", transform it: rewrite as an invariant, checklist item, failure signal, or example. If it cannot be transformed into any of those forms, delete it.

**Form transformation test**: "The skill must teach judgment well" → hollow. "Stop and revise if the trigger enumerates scenarios instead of describing a state" → high-signal.
