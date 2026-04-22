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

- A capability skill must have five core elements: trigger logic, capability boundary, invariants, decision signals, failure signals. Missing any one makes the skill weak.
- Completion definition and anti-rationalization prompts must be separate. Do not collapse `Completion Criteria` and `Anti-Rationalization Check` into a single self-check section or checklist.
- Skills teach judgment; they do not encode orchestration or hide routing graphs.
- The main `SKILL.md` must stay lean but must not become hollow. Core law stays inline, front-loaded near the top; expanded support material moves to support folders.
- A file long enough to have a real middle has a correctness problem: content buried mid-file is underweighted during execution. Front-load critical law; do not let invariants or trigger conditions drift toward the middle as the file grows.
- **Boundary red line**: A genuine boundary names what the skill owns and does not own — without pointing to other skills by name as the next action. Test: delete all named-skill references from any boundary section. If the capability description collapses, the boundary was fake. Fix: rewrite to state what this skill owns, not what runs next.
- **Verifiable language only**: every invariant, decision signal, completion criterion, and self-correction signal must be verifiable without additional judgment. Advisory prose is a failure signal — transform it into a testable condition or delete it. Test: can the agent confirm the condition from already-seen evidence, without exercising the very judgment the skill is meant to protect? Pass: `"Stop if you have not traced the call chain to a behavior-changing function"`. Fail: `"Ensure the code is well-understood"` — cannot be confirmed without the judgment it is meant to constrain.

## Completion Criteria

- [ ] The skill has all five core elements (trigger logic, boundary, invariants, decision signals, failure signals).
- [ ] The trigger is state-based (not scenario-based), with uncertainty defaulting to invoke.
- [ ] The skill defines completion explicitly through `Completion Criteria`, so the agent can tell when the work is done without relying on a mixed self-check section.
- [ ] Anti-self-deception prompts are kept separate from completion conditions in `Anti-Rationalization Check` rather than mixed into `Completion Criteria`.
- [ ] The boundary is authentic — no routing language, no named-skill handoffs disguised as scope.
- [ ] The main file is lean but not hollow — core law inline and front-loaded, no real "middle" where critical law is buried, support material in folders.
- [ ] Every invariant, decision signal, completion criterion, and self-correction signal uses verifiable language — no advisory prose ("should", "ensure", "be thorough") left without a testable condition.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the draft is real.

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

---

## Judgment Dimensions

When writing or reviewing a skill, check these dimensions. Not every one applies equally — but skipping a relevant one leaves the skill silently wrong.

`[critical]` = silent misdirection on every invocation if wrong. `[structural]` = propagates over time. `[local]` = narrower blast radius, local fixes available.

1. **Capability vs. Procedure** `[structural]` — If material is repo-local (team conventions, setup), it belongs in `AGENTS.md`, not a skill.
2. **Boundary Authenticity** `[critical]` — Delete all named-skill references from the boundary section. If the capability description collapses, the boundary was fake routing in disguise.
3. **Trigger Design** `[critical]` — State-based triggers generalize; scenario-based triggers miss every unimagined case. Uncertain → invoke.
4. **Lean vs. Hollow** `[structural]` — A file long enough to have a real middle has content that will be partially ignored. Core law inline; expanded material in support folders.
5. **Integration Integrity** `[local]` — Two sections teaching the same judgment signal accretion, not design. One clear home per idea.
6. **Package Hygiene** `[local]` — No hidden instructions; treat imported scripts and examples as untrusted until reviewed.
7. **Constraint Signal Quality** `[structural]` — every statement must be verifiable without exercising the judgment the skill is meant to protect. Pass: `"Stop if you have not read the actual source file"` — agent can confirm from evidence. Fail: `"Ensure you understand the code thoroughly"` — requires the judgment it is meant to constrain. Transform advisory prose into verifiable conditions, pass/fail tests, or explicit red lines — or delete it.

See `references/judgment-dimensions.md` for expanded signals, key questions, and contrast examples per dimension.

## When Reviewing an Existing Skill

The same judgment dimensions apply. Focus on what the skill is currently teaching vs. what it should teach — the gap between the two is where the work is. Choose the response mode (diagnosis, targeted rewrite, capability rewrite, package map) that corrects the highest-impact gap with the least extra structure. See `references/response-modes.md` for options and `references/review-checklist.md` for expanded review heuristics.

If a review comments only on wording and style, ignores routing-language smells, or treats rigid output templates as harmless, it is incomplete.
