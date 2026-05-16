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

## Principles

- A capability skill must have six core elements: trigger conditions, boundary and non-goals, task contract, completion check, examples, and failure modes. Missing any one makes the skill weak.
- **Design for unreliable introspection**: a skill exists because model self-awareness is limited and unstable — a model cannot reliably notice when it is guessing, rationalizing, or about to skip a step. The purpose of invariants, stop conditions, completion checks, and failure signals is to externalize those checkpoints, not to remind the model to be more careful. A skill that says "think carefully before acting" has failed this invariant. A skill that says "before acting, confirm you have read the affected call sites" has succeeded.
- A completion check question must be answerable from the edited skill or conversation evidence. A question that requires the agent to judge its own work quality is advisory prose in disguise.
- Skills teach judgment; they do not encode orchestration or hide routing graphs [because explicit skill handoffs can mask that the routed skill's trigger conditions are too broad, too narrow, or unclear; fix the routed skill's trigger instead of teaching another skill to route to it].
- The main `SKILL.md` must follow progressive disclosure: keep the rules an agent must see to invoke and execute the skill in the main file, and move expanded explanation, long examples, templates, and reference material to support folders [because agents read skills under limited attention, so the first file must expose the decision-critical law without forcing a deep reference chase].
- **Remove inducements before adding counter-rules**: when an existing skill causes unwanted behavior because its wording, examples, or workflow make that behavior the obvious move, first name the inducing source and delete or neutralize it [because adding a counter-rule can keep the unwanted behavior salient and make the skill revolve around the behavior it should stop teaching]. Add a negative boundary only when the behavior remains a real scope edge after the inducement is removed and the boundary can name an observable wrong-action condition.
- **Inline rationale rule**: not every element needs a "why," but some elements require it. Required: trigger conditions (agent must understand what problem is skipped to judge new scenarios), invariants (rules without reasons get rationalized away in edge cases), failure modes (agent that knows to stop but not why will repeat the pattern in a new form). Partial — non-obvious cases only: boundary non-goals, completion checks. Not required: examples (purpose is universal). Test: would an agent in a new scenario correctly apply this rule without knowing why it exists? If no → add inline rationale. Format: append the reason as a clause — "never add abstraction without a second caller [because premature abstraction creates coupling without evidence of need]."

## Completion Check

Before accepting a skill creation or optimization, answer these questions from the edited skill or conversation evidence. If any answer is no, or cannot be answered from evidence, revise the skill.

Can an agent decide whether to use the skill from the trigger and boundary without opening support files first?

Does the main `SKILL.md` contain the decision-critical rules needed to execute the skill, with support files reserved for expanded examples, templates, references, scripts, or assets?

Does the skill teach reusable agent judgment instead of fixed orchestration, hidden skill routing, or project-local procedure?

Are invariants, decision signals, completion checks, and failure signals verifiable from observable evidence rather than advisory prose?

Do examples or support-file pointers cover both positive use and boundary/failure cases?

Did you leave any trigger, boundary, invariant, example, or checklist item that would make an agent use this skill outside skill creation or skill optimization?

Did you add a `Do not use` rule without naming the concrete wrong-invocation behavior it prevents?

Did you name another skill as the next action instead of fixing that skill's own trigger or boundary?

Did you add a counter-rule while leaving the wording, example, or workflow that induced the bad behavior in place?

Did you move core trigger, boundary, invariant, completion, or failure-mode logic into support files so the main `SKILL.md` is no longer usable on first read?

Did you add a new section where strengthening an existing section would teach the same judgment with less structure?

Did you leave two sections teaching the same judgment in different words?

Did you leave advisory prose where an observable condition or explicit red line is needed?

Did you accept completion checks that test format or section presence instead of whether the skill changes agent behavior in the intended direction?

Are you accepting the skill because all sections exist, or because the edited skill would actually guide future agents toward the intended behavior?

---

See `references/judgment-dimensions.md` for judgment dimensions, key questions, and contrast examples when writing or reviewing a skill.
See `examples/seven-elements-examples.md` for positive and negative examples of each core element.
See `examples/progressive-disclosure-examples.md` when deciding what stays in the main `SKILL.md` versus what moves to examples or references.
