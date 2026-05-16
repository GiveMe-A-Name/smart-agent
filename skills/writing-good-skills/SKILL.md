---
name: writing-good-skills
description: "Write capability skills that teach reusable agent judgment through boundaries, causal rules, and calibrated examples. TRIGGER when creating a new skill or optimizing an existing skill."
---

# Writing Good Skills

Write skills as reusable capability guides that teach judgment

A skill is evaluated for judgment quality, not format compliance. 
The value is in the boundary: understanding what the skill owns, what must not happen, what signals should shape decisions, and where the capability ends. A well-written skill still guides an agent when followed out of order, partially read, or applied to a case its author never imagined. 
A well-written SOP is not a failure. The failure is using SOPs for judgment tasks: a fixed workflow freezes the agent's choices before it has seen the case, replacing context-sensitive judgment with step execution and blocking simpler, more accurate, or more adaptive solutions.

## The Reader Is an Agent

A SKILL.md is read and executed by another agent — not a human. This single fact constrains every language choice. Two consequences follow, and most of the invariants below are downstream applications of these two.

**Agents do not judge by atmosphere.**

Words like "ensure", "be thorough", "carefully consider", "as appropriate", "well-understood" delegate the check back to the reading agent's own judgment. But the reason the skill exists is that the agent's self-judgment is unreliable in exactly this situation — otherwise no skill would be needed. Atmosphere language asks the agent to exercise the very capability the skill was written to compensate for, which is why it silently fails.

Every condition the agent must verify — invariants, decision signals, completion criteria, failure signals — must be confirmable from observable evidence already seen: call sites read, tests run, files opened, user statements made.

Test: can the reading agent answer yes/no to this condition without exercising the very judgment the skill is meant to protect?
- Pass: "Stop if you have not traced the call chain to a behavior-changing function" — checkable against actions taken.
- Fail: "Ensure the code is well-understood" — only the unreliable self-judgment can answer.

**Agents generalize from causal models, not from rules or examples alone.**

A skill is invoked in situations its author never imagined. An agent that knows a rule but not the mechanism behind it will fail to apply it in a structurally identical case that does not match the surface pattern — and will rationalize its way around the rule when the surface pattern looks different enough.

Every trigger condition, invariant, and failure mode that could plausibly meet a novel scenario must include the mechanism — the chain from "if this happens" to "this specific kind of downstream harm follows". Examples illustrate the mechanism; they do not substitute for it.

Test: shown only the rule and its stated reason, could the reading agent derive the right action in a scenario the author never imagined? If the right action requires "getting the spirit" of the rule, the causal model is missing — write it inline.

These two consequences are why the verifiable-language and inline-rationale rules below exist. When in doubt about a specific phrasing, return here.

## Trigger Logic

**Invocation default**: Use this skill only when the task is to create a skill or optimize an existing skill. Do not broaden the task into documentation-routing or knowledge-placement triage [because that makes the skill fire on generic documentation decisions instead of skill work].

Use this skill when:
- creating a new skill from notes, habits, workflows, or a fresh idea [because the skill needs trigger conditions, boundaries, invariants, completion criteria, verification, examples, and failure modes before it can guide future agents]
- optimizing an existing skill's trigger conditions, boundaries, invariants, examples, completion criteria, verification approach, or observed behavior [because small wording changes can alter when agents invoke the skill or what behavior it teaches]

Do not use this skill when:
- the task is deciding whether arbitrary material belongs in a skill, `AGENTS.md`, `README.md`, a note, or another documentation surface, unless the user has explicitly framed it as skill creation or skill optimization [because documentation-placement triage is broader than this capability]
- the change is confirmed to be a wording fix only, with no effect on trigger conditions, boundaries, invariants, examples, completion criteria, verification approach, or judgment [because this skill adds no value when skill behavior cannot change]

## Boundary

Use this skill when the requested work should make a skill produce more reliable agent behavior: the skill should trigger in the right situations, stay silent outside its capability, externalize the checks agents cannot reliably self-assess, and teach reusable judgment rather than a fixed procedure. If the requested change does not affect those behavior-shaping properties, this skill is probably not the right tool.

This skill improves a skill by:
- converting notes, habits, or workflows into reusable capability guidance with observable trigger conditions [because future agents need evidence for invocation, not author intent]
- sharpening trigger conditions and non-goals so the skill fires only for its capability [because broad triggers make unrelated tasks inherit the wrong workflow]
- replacing advisory prose with verifiable invariants, completion criteria, and self-correction signals [because vague reminders delegate the check back to model self-judgment]
- separating capability guidance from orchestration, routing language, and project-local procedures [because reusable skills must transfer across cases instead of hard-coding one workflow]
- keeping critical law inline and front-loaded while moving only supporting detail out of the main file [because buried rules are underweighted during execution]
- adding positive and boundary examples that calibrate where the skill applies [because positive-only examples make agents overgeneralize the capability]

This skill does not improve a skill by:
- inventing or validating the skill's domain content — this skill shapes how domain knowledge is taught, not whether that knowledge is true
- deciding where arbitrary project knowledge belongs across `AGENTS.md`, `README.md`, notes, or other documentation surfaces
- creating application-specific workflows or project-local operating rules
- evaluating whether the underlying task or workflow is correct
- making wording-only edits that cannot change when the skill triggers or what behavior it teaches


## Invariants

- A capability skill must have six core elements: trigger conditions, boundary and non-goals, task contract, completion criteria, examples, and failure modes. Missing any one makes the skill weak.
- **Design for unreliable introspection**: a skill exists because model self-awareness is limited and unstable — a model cannot reliably notice when it is guessing, rationalizing, or about to skip a step. The purpose of invariants, stop conditions, completion criteria, and failure signals is to externalize those checkpoints, not to remind the model to be more careful. A skill that says "think carefully before acting" has failed this principle. A skill that says "before acting, confirm you have read the affected call sites" has succeeded.
- When a completion criterion is a state condition, it must be framed as a yes/no question answerable from conversation evidence. A state condition that requires the agent to judge its own work quality is not a state condition — it is advisory prose in disguise. Whether a skill uses state conditions or artifact conditions depends on skill type (see Verification Approach).
- Skills teach judgment; they do not encode orchestration or hide routing graphs [because explicit skill handoffs can mask that the routed skill's trigger conditions are too broad, too narrow, or unclear; fix the routed skill's trigger instead of teaching another skill to route to it].
- The main `SKILL.md` must follow progressive disclosure: keep the rules an agent must see to invoke and execute the skill in the main file, and move expanded explanation, long examples, templates, and reference material to support folders [because agents read skills under limited attention, so the first file must expose the decision-critical law without forcing a deep reference chase].
- **Remove inducements before adding counter-rules**: when an existing skill causes unwanted behavior because its wording, examples, or workflow make that behavior the obvious move, first name the inducing source and delete or neutralize it [because adding a counter-rule can keep the unwanted behavior salient and make the skill revolve around the behavior it should stop teaching]. Add a negative boundary only when the behavior remains a real scope edge after the inducement is removed and the boundary can name an observable wrong-action condition.
- **Inline rationale rule**: not every element needs a "why," but some elements require it. Required: trigger conditions (agent must understand what problem is skipped to judge new scenarios), invariants (rules without reasons get rationalized away in edge cases), failure modes (agent that knows to stop but not why will repeat the pattern in a new form). Partial — non-obvious cases only: boundary non-goals, completion criteria. Not required: examples (purpose is universal). Test: would an agent in a new scenario correctly apply this rule without knowing why it exists? If no → add inline rationale. Format: append the reason as a clause — "never add abstraction without a second caller [because premature abstraction creates coupling without evidence of need]."

## Completion Criteria

- [ ] The skill's trigger and boundary are enough for an agent to decide whether to use the skill without opening support files first.
- [ ] The main `SKILL.md` contains the decision-critical rules needed to execute the skill; support files are only needed for expanded examples, templates, references, or scripts.
- [ ] The skill teaches reusable agent judgment instead of fixed orchestration, hidden skill routing, or project-local procedure.
- [ ] Invariants, decision signals, completion criteria, and self-correction signals are verifiable from observable evidence rather than advisory prose.
- [ ] Required causal rationale is inline for triggers, non-obvious invariants, and failure modes, so agents can apply the rule in new cases.
- [ ] Examples or support-file pointers cover both positive use and boundary/failure cases.
- [ ] Completion criteria make the exit condition observable from the edited skill or conversation state.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

This section exists because model introspection is unreliable when a skill draft looks complete. Use these questions to confirm the product judgment, not only readiness to exit: the edited skill must actually teach the intended behavior.

Ask before exiting:

Did you leave any trigger, boundary, invariant, example, or checklist item that would make an agent use this skill outside skill creation or skill optimization?

Did you add a `Do not use` rule without naming the concrete wrong-invocation behavior it prevents?

Did you name another skill as the next action instead of fixing that skill's own trigger or boundary?

Did you add a counter-rule while leaving the wording, example, or workflow that induced the bad behavior in place?

Did you move core trigger, boundary, invariant, completion, or failure-mode logic into support files so the main `SKILL.md` is no longer usable on first read?

Did you add a new section where strengthening an existing section would teach the same judgment with less structure?

Did you leave two sections teaching the same judgment in different words?

Did you leave advisory prose where an observable condition or explicit red line is needed?

Did you accept completion criteria that test format or section presence instead of whether the skill changes agent behavior in the intended direction?

Are you accepting the skill because all sections exist, or because the edited skill would actually guide future agents toward the intended behavior?

---

See `references/judgment-dimensions.md` for judgment dimensions, key questions, and contrast examples when writing or reviewing a skill.
See `examples/seven-elements-examples.md` for positive and negative examples of each core element.
See `examples/progressive-disclosure-examples.md` when deciding what stays in the main `SKILL.md` versus what moves to examples or references.

## When Reviewing an Existing Skill

The same judgment dimensions apply. Focus on what the skill is currently teaching vs. what it should teach — the gap between the two is where the work is. Choose the response mode (diagnosis, targeted rewrite, capability rewrite, package map) that corrects the highest-impact gap with the least extra structure. See `references/response-modes.md` for options and `references/review-checklist.md` for expanded review heuristics.

If a review comments only on wording and style, ignores routing-language smells, or treats rigid output templates as harmless, it is incomplete.
