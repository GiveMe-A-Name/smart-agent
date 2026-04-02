---
name: writing-good-skills
description: "Invoke when creating a skill, modifying a SKILL.md, or when a skill's behavior diverges from expectations. Cost of unnecessary invocation: a brief structural review. Cost of missing: a skill that silently triggers at the wrong time, encodes routing instead of capability, or teaches the wrong behavior while appearing complete. When uncertain whether a change affects trigger conditions or boundaries, invoke."
---

# Writing Good Skills

Write skills as reusable capability guides that teach judgment — not SOPs, procedure manuals, or hidden routing graphs.

A skill is evaluated for judgment quality, not format compliance. The value is in the boundary: understanding what the skill owns, what must not happen, what signals should shape decisions, and where the capability ends. A well-written skill still guides an agent when followed out of order, partially read, or applied to a case its author never imagined. A well-written SOP is not a failure — the failure is forcing judgment tasks into SOPs, or dressing up SOPs as capability skills.

## Trigger Logic

**Invocation default**: The cost of running this skill unnecessarily is a short review pass. The cost of skipping it is a skill that silently triggers at the wrong time, encodes routing instead of capability, or teaches the wrong thing while looking complete. When skill quality or structure is in question, invoke.

Use this skill when:
- creating a skill from notes, habits, or a fresh idea
- modifying any `SKILL.md`, including when a skill's behavior diverges from what is expected
- deciding whether material belongs in a skill, `AGENTS.md`, `README.md`, or a note

Do not use this skill when the change is confirmed to be a wording fix only, with no effect on trigger conditions, boundaries, invariants, or judgment.

## Boundary

This skill owns:
- deciding whether material qualifies as a reusable capability vs. local documentation
- judging whether a draft teaches judgment or encodes orchestration
- identifying routing language, trigger design failures, and lean-vs-hollow tradeoffs
- deciding what belongs in the main file vs. support folders

This skill does not own:
- writing the skill's domain content — this skill teaches structure, not substance
- creating application-specific workflows or project-local operating rules
- evaluating whether the underlying task or workflow is correct


## Invariants

- A capability skill must have five core elements: trigger logic, capability boundary, invariants, decision signals, failure signals. Missing any one makes the skill weak.
- Completion definition and anti-rationalization prompts must be separate. Do not collapse `Completion Criteria` and `Anti-Rationalization Check` into a single self-check section or checklist.
- Skills teach judgment; they do not encode orchestration or hide routing graphs.
- The main `SKILL.md` must stay lean but must not become hollow. Core law stays inline; expanded support material moves to support folders.

## Completion Criteria

- [ ] The skill has all five core elements (trigger logic, boundary, invariants, decision signals, failure signals).
- [ ] The trigger is state-based (not scenario-based), with uncertainty defaulting to invoke.
- [ ] The skill defines completion explicitly through `Completion Criteria`, so the agent can tell when the work is done without relying on a mixed self-check section.
- [ ] Anti-self-deception prompts are kept separate from completion conditions in `Anti-Rationalization Check` rather than mixed into `Completion Criteria`.
- [ ] The boundary is authentic — no routing language, no named-skill handoffs disguised as scope.
- [ ] The main file is lean but not hollow — core law inline, support material in folders.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the draft is real.

Did I ignore any applicable self-correction signals because the draft already looked good?

Am I exiting because the skill is genuinely well-designed, or because the structure now looks complete enough?

## Judgment Dimensions

When writing or reviewing a skill, work through these dimensions. Not every dimension applies equally to every skill — but skipping a relevant one leaves the skill silently wrong.

### 1. Capability vs. Procedure Character

**Question**: Does this material need a capability skill, or would an SOP, a note, or local documentation serve it better?

**Key signals**:
- Procedural tasks have fixed steps and low variation — the same sequence applies every time regardless of context. These are fine as SOPs. Forcing them into the five-element capability framework adds ceremony without improving judgment.
- Judgment tasks have high variation, context-dependent paths, or decisions that only make sense after reading the situation. These need capability skills.
- If the material is mostly repo-local context — team conventions, setup instructions, project-specific rules — it belongs in `AGENTS.md` or `README.md`, not in a reusable skill.
- When input is raw notes or a workflow description, the most common failure is renumbering the notes into steps rather than extracting the underlying judgment. The invariants, boundary conditions, and decision signals are usually buried in the notes — extracting them is the work.
- When creating a skill from scratch, make boundary, trigger, and invariants decisions explicitly before drafting the body. Do not wait for a draft to surface naturally from the input.

See `examples/notes-to-skill.md` for a before/after showing what goes wrong when notes become renumbered steps.

### 2. Boundary Authenticity

**Question**: Does the draft state what the skill owns, or is it secretly a routing graph?

**Key signals**:
- Routing language answers: *what runs next?* Boundary language answers: *what does this skill own, and what is missing when it cannot proceed?*
- A genuine boundary names what the skill owns and does not own, without pointing to other skills by name as the next action.
- Boundary test: delete all named-skill references from the draft. If the capability description collapses, the boundary was fake — it was doing routing work dressed as scope definition.
- `Relationship to...` sections are acceptable only when they clarify scope — stating what this skill does not cover. They must not prescribe what to invoke next.
- When a section names another specific skill as something to invoke in a given scenario, the named skill's trigger conditions are too weak to fire on their own. The fix belongs in the named skill's trigger logic — not here.

See `examples/routing-vs-boundary.md` for a contrast pair.

### 3. Trigger Design

**Question**: Does the trigger describe a state, or enumerate scenarios that will miss cases not yet imagined?

**Key signals**:
- State-based triggers fire on what is true about the situation: "code is about to be written," "skill behavior is in question." These generalize.
- Scenario-based triggers fire on specific patterns: "user pastes X," "output looks like Y." These miss every unimagined case.
- Uncertain → invoke is the right default. The cost asymmetry is clear: unnecessary invocation costs a short reasoning pass; missed invocation costs downstream errors that are hard to recover. State this asymmetry explicitly in the Trigger Logic section.
- `Do not use` conditions are only necessary when there is a realistic wrong-invocation scenario the positive trigger state does not already exclude. Before adding one, name the specific wrong invocation it prevents. If the answer is vague, delete the condition.
- `Do not use` conditions placed in the description face maximum rationalization pressure — they are evaluated before the agent has invested in the situation. Prefer moving them to the Trigger Logic body.

See `references/skill-trigger-design-principles.md` for the full trigger design framework.
See `references/description-patterns.md` for the description-field and invocation-default patterns.
See `references/thinking-mode-considerations.md` for how these principles apply in thinking-mode agents.

### 4. Lean vs. Hollow

**Question**: Is the main file lean because support folders carry what belongs there, or hollow because core law was moved out?

**Key signals**:
- Lean: the main file contains all five core elements; support folders carry expanded examples, lookup tables, and templates.
- Hollow: trigger logic, invariants, or core decision signals were moved to support files, leaving an agent who must hunt through the package to find the actual law.
- Test: can the main file guide a first-pass decision without opening any support file? If not, something core was moved out.
- Support folders by function: `references/` for lookup material and review heuristics; `templates/` for reusable scaffolds; `examples/` for realistic bad-vs-good contrast pairs; `scripts/` for deterministic helpers.
- Add an `examples/` folder only when a bad pattern looks superficially similar to a good one and prose rules alone won't prevent the wrong output. A good example pair: a realistic bad case alongside a corrected version, annotated to name what makes it wrong — not a restatement of rules the prose already makes clear.

See `references/lean-skill-patterns.md` for split rules and folder taxonomy.

### 5. Integration Integrity

**Question**: Does each important idea have one clear home?

**Key signals**:
- The same rule appearing in multiple sections signals the skill was built by accretion, not by judgment.
- Before adding a new section, identify whether the idea already has a home elsewhere. Strengthen the existing home first.
- Stop and compress when: two sections teach the same judgment in slightly different words; an example repeats what bullets already say; a section exists only to defend a rule that is already clear.
- Decision order: merge into the strongest existing section → tighten wording → replace duplication with a cross-reference → add a new section only if the idea adds a new decision signal, invariant, or failure signal.
- Shorter is not the goal. One clear home per idea is the goal.

### 6. Package Hygiene

**Question**: Does the package teach the right things without hidden instructions or unsafe imports?

**Key signals**:
- Do not include hidden instructions or prompt-injection style content in the skill package.
- Treat imported scripts, examples, and third-party references as untrusted until reviewed.
- Keep tools, scripts, and assets to the minimum the skill actually needs.
- Do not normalize unsafe data handling just because it appears in an example.


## When Reviewing an Existing Skill

The same judgment dimensions apply. Focus on what the skill is currently teaching vs. what it should teach — the gap between the two is where the work is. Choose the response mode (diagnosis, targeted rewrite, capability rewrite, package map) that corrects the highest-impact gap with the least extra structure. See `references/response-modes.md` for options and `references/review-checklist.md` for expanded review heuristics.

If a review comments only on wording and style, ignores routing-language smells, or treats rigid output templates as harmless, it is incomplete.


## Self-Correction Signals

Stop and revise when:
- the trigger enumerates specific scenarios instead of describing a state
- a `Do not use` condition was added reflexively without naming a concrete wrong-invocation scenario
- the draft names a specific skill as something to invoke — this signals the named skill's trigger needs fixing, not that this skill should encode the handoff
- the draft reads like a renumbered version of the input notes or workflow description
- core law was moved to support files, leaving the main file hollow
- a new section was added where strengthening an existing section would have done the same work
- removing named-skill references leaves the capability description empty or incoherent
- two sections teach the same judgment in slightly different words


