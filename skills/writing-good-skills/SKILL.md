---
name: writing-good-skills
description: "Guide skill creation and modification — structure, trigger conditions, boundaries. TRIGGER when: creating a skill, modifying a SKILL.md, or a skill's behavior diverges from expectations. DO NOT TRIGGER when: using a skill normally or working on non-skill files."
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

- A capability skill must have seven core elements: trigger conditions, boundary and non-goals, task contract, completion criteria, verification approach, examples, and failure modes. Missing any one makes the skill weak.
- **Design for unreliable introspection**: a skill exists because model self-awareness is limited and unstable — a model cannot reliably notice when it is guessing, rationalizing, or about to skip a step. The purpose of invariants, stop conditions, completion criteria, and failure signals is to externalize those checkpoints, not to remind the model to be more careful. A skill that says "think carefully before acting" has failed this principle. A skill that says "before acting, confirm you have read the affected call sites" has succeeded.
- A skill is mode-type, not function-type. The task contract — typically the `Invariants` section — describes rules that hold throughout the interaction, not an input/output spec. Test: if the contract reads like a function signature (input X → output Y), it is wrong. Rewrite to describe what must remain true across all turns.
- When a completion criterion is a state condition, it must be framed as a yes/no question answerable from conversation evidence. A state condition that requires the agent to judge its own work quality is not a state condition — it is advisory prose in disguise. Whether a skill uses state conditions or artifact conditions depends on skill type (see Verification Approach).
- Completion definition and anti-rationalization prompts must be separate. Do not collapse `Completion Criteria` and `Anti-Rationalization Check` into a single self-check section or checklist.
- Skills teach judgment; they do not encode orchestration or hide routing graphs.
- The main `SKILL.md` must stay lean but must not become hollow. Core law stays inline, front-loaded near the top; expanded support material moves to support folders.
- A file long enough to have a real middle has a correctness problem: content buried mid-file is underweighted during execution. Front-load critical law; do not let invariants or trigger conditions drift toward the middle as the file grows.
- **Boundary red line**: A genuine boundary names what the skill owns and does not own — without pointing to other skills by name as the next action. Test: delete all named-skill references from any boundary section. If the capability description collapses, the boundary was fake. Fix: rewrite to state what this skill owns, not what runs next.
- **Proxy metric risk**: structural completeness ≠ skill effectiveness [because a skill can satisfy every checklist item while still being hollow]. A skill with all seven elements present, verifiable criteria, and examples fails this test if: invariants contain advisory prose, completion criteria test section presence rather than judgment quality, failure signals list patterns without rationale. Test: could an agent game every checklist item in this skill's Completion Criteria while producing a skill that teaches nothing? If yes, the criteria are measuring the wrong thing — rewrite them to test substance, not format.
- **Inline rationale rule**: not every element needs a "why," but some elements require it. Required: trigger conditions (agent must understand what problem is skipped to judge new scenarios), invariants (rules without reasons get rationalized away in edge cases), failure modes (agent that knows to stop but not why will repeat the pattern in a new form). Partial — non-obvious cases only: boundary non-goals, completion criteria. Not required: verification approach (mechanism is self-evident), examples (purpose is universal). Test: would an agent in a new scenario correctly apply this rule without knowing why it exists? If no → add inline rationale. Format: append the reason as a clause — "never add abstraction without a second caller [because premature abstraction creates coupling without evidence of need]."
- **Verifiable language only**: every invariant, decision signal, completion criterion, and self-correction signal must be verifiable without additional judgment [because advisory prose delegates the check back to the model's self-awareness — exactly what the skill exists to replace]. Advisory prose is a failure signal — transform it into a testable condition or delete it. Test: can the agent confirm the condition from already-seen evidence, without exercising the very judgment the skill is meant to protect? Pass: `"Stop if you have not traced the call chain to a behavior-changing function"`. Fail: `"Ensure the code is well-understood"` — cannot be confirmed without the judgment it is meant to constrain.

## Completion Criteria

- [ ] The skill has all seven core elements: trigger conditions, boundary and non-goals, task contract, completion criteria, verification approach, examples, and failure modes.
- [ ] The trigger is state-based (not scenario-based), with uncertainty defaulting to invoke.
- [ ] The skill defines completion explicitly through `Completion Criteria`, so the agent can tell when the work is done without relying on a mixed self-check section.
- [ ] Anti-self-deception prompts are kept separate from completion conditions in `Anti-Rationalization Check` rather than mixed into `Completion Criteria`.
- [ ] The boundary is authentic — no routing language, no named-skill handoffs disguised as scope.
- [ ] The main file is lean but not hollow — core law inline and front-loaded, no real "middle" where critical law is buried, support material in folders.
- [ ] Every invariant, decision signal, completion criterion, and self-correction signal uses verifiable language — no advisory prose ("should", "ensure", "be thorough") left without a testable condition.
- [ ] Every completion criterion is either: (a) an artifact condition checkable against evidence in the output, or (b) a state condition framed as a yes/no question answerable from conversation evidence. Test: apply "The skill teaches good judgment" — fails (a) because no artifact to check, fails (b) because not answerable with yes/no from evidence. Delete or rewrite any criterion that fails both tests.
- [ ] Trigger conditions include inline rationale stating what problem occurs if the trigger is missed. Failure modes include inline rationale stating why the pattern leads to failure — not just that it does. Non-obvious invariants and boundary non-goals include inline rationale; self-evident ones do not.
- [ ] The skill includes at least one positive example and one negative or boundary example. A skill with positive examples only does not anchor agent judgment on where the line is.
- [ ] The verification approach is explicit: either "self-check against this checklist" (artifact-type) or "user confirms in dialogue" (dialogue-outcome-type). If unspecified, agent cannot distinguish between "I think I'm done" and "I am done."

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is artifact-type: completion is verified by self-checking the output against the Completion Criteria checklist above. No separate user confirmation is required.

**Skill type determines verification method.** Two types exist:
- **Artifact-type** (plan docs, code, skill files): completion is self-checked against a checklist. User confirmation is optional, or embedded in the artifact itself (e.g., a Human Review Section).
- **Dialogue-outcome-type** (brainstorming, direction-setting): completion requires user confirmation in dialogue — the outcome is a shared understanding that cannot be self-verified.

When writing a skill's Completion Criteria: state which type it is and what verification looks like. An unspecified verification approach leaves the agent unable to distinguish "I believe I'm done" from "I am done."

## Anti-Rationalization Check

This section exists because model introspection is unreliable in exactly the moment it matters most — when the work looks finished. The model's tendency to produce plausible-sounding completions means a skill draft that looks structurally sound may still encode advisory prose, fake boundaries, or hollow criteria. This check is not a personal honesty prompt; it is a compensating mechanism for that instability.

Pause before exiting. Do not treat this section as another checklist to clear.

Did I ignore any applicable self-correction signals because the draft already looked good?

Am I exiting because the skill is genuinely well-designed, or because the structure now looks complete enough?

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
- critical trigger conditions or invariants are buried mid-file rather than front-loaded near the top
- a single section lists many independent constraints without a unifying principle or priority ordering
- an invariant or decision signal reads like advisory prose ("should", "ensure", "be thorough") rather than a verifiable condition or explicit red line — transform it or delete it
- a failure mode states what to stop for but not why — an agent that does not understand the reason will repeat the same pattern in a different form and not recognize it as the same failure
- completion criteria test format rather than substance — "Invariants section has at least three items" passes a skill with three advisory-prose invariants; "skill has an examples folder" passes a skill with two positive-only examples; rewrite criteria to test whether the element does its job, not whether it exists
- the skill satisfies its own Completion Criteria but the checklist items could all be gamed while producing a hollow skill — this is proxy metric failure; stop and rewrite the criteria to close the gap between formal pass and real effectiveness

---

See `references/judgment-dimensions.md` for judgment dimensions, key questions, and contrast examples when writing or reviewing a skill.
See `examples/seven-elements-examples.md` for positive and negative examples of each core element.

## When Reviewing an Existing Skill

The same judgment dimensions apply. Focus on what the skill is currently teaching vs. what it should teach — the gap between the two is where the work is. Choose the response mode (diagnosis, targeted rewrite, capability rewrite, package map) that corrects the highest-impact gap with the least extra structure. See `references/response-modes.md` for options and `references/review-checklist.md` for expanded review heuristics.

If a review comments only on wording and style, ignores routing-language smells, or treats rigid output templates as harmless, it is incomplete.
