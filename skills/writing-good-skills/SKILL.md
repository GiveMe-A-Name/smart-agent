---
name: writing-good-skills
description: "Write capability skills that teach reusable agent judgment through principles, constraints, and calibrated examples. TRIGGER when creating a new skill or optimizing an existing skill."
---

# Writing Good Skills

Write skills as reusable capability guides that teach judgment

A skill is evaluated for judgment quality, not format compliance. 
The value is in the operating model: understanding what principles should shape decisions, what constraints must hold, and where the capability ends. A well-written skill still guides an agent when followed out of order, partially read, or applied to a case its author never imagined.
A well-written SOP is not a failure. The failure is using SOPs for judgment tasks: a fixed workflow freezes the agent's choices before it has seen the case, replacing context-sensitive judgment with step execution and blocking simpler, more accurate, or more adaptive solutions.

## The Reader Is an Agent

A SKILL.md is read and executed by another agent — not a human. This single fact constrains every language choice. Two consequences follow, and the Principles and Constraints below apply them.

**Agents do not judge by atmosphere.**

Words like "ensure", "be thorough", "carefully consider", "as appropriate", "well-understood" delegate the check back to the reading agent's own judgment. But the reason the skill exists is that the agent's self-judgment is unreliable in exactly this situation — otherwise no skill would be needed. Atmosphere language asks the agent to exercise the very capability the skill was written to compensate for, which is why it silently fails.

Every constraint the agent must verify — boundary, evidence requirement, red line, or stop condition — must be confirmable from observable evidence already seen: call sites read, tests run, files opened, user statements made.

Test: can the reading agent answer yes/no to this condition without exercising the very judgment the skill is meant to protect?
- Pass: "Stop if you have not traced the call chain to a behavior-changing function" — checkable against actions taken.
- Fail: "Ensure the code is well-understood" — only the unreliable self-judgment can answer.

**Agents generalize from causal models, not from rules or examples alone.**

A skill is invoked in situations its author never imagined. An agent that knows a rule but not the mechanism behind it will fail to apply it in a structurally identical case that does not match the surface pattern — and will rationalize its way around the rule when the surface pattern looks different enough.

Every frontmatter load condition, principle, or constraint that must generalize to novel scenarios must include the mechanism — the chain from "if this happens" to "this specific kind of downstream harm follows". Examples illustrate the mechanism; they do not substitute for it.

Test: shown only the rule and its stated reason, could the reading agent derive the right action in a scenario the author never imagined? If the right action requires "getting the spirit" of the rule, the causal model is missing — write it inline.

These two consequences are why the verifiable-language and inline-rationale rules below exist. When in doubt about a specific phrasing, return here.

## Principles

- A capability skill is built from two primary parts: **Principles** and **Constraints**. Principles guide the agent toward the intended judgment and action; constraints define boundaries, non-goals, red lines, and observable stop conditions that keep the agent from doing the wrong work [because agents do not carry stable task-specific judgment or reliable self-awareness across contexts: principles provide the positive policy they can generalize from, while constraints externalize the negative checks that prevent rationalization, scope drift, and unverifiable self-judgment].
- **Design for unreliable introspection**: handle weak model self-awareness through the two-part structure only. Principles state the causal policy the agent should follow; constraints state observable boundaries, evidence requirements, red lines, and stop conditions. A principle that says "think carefully before acting" has failed because it relies on self-judgment; a constraint that says "before acting, confirm you have read the affected call sites" has succeeded because it replaces self-judgment with evidence.
- Skills teach judgment; they do not encode orchestration or hide routing graphs [because explicit skill handoffs can mask that the routed skill's frontmatter `description` is too broad, too narrow, or unclear; fix that description instead of teaching another skill to route to it].
- The main `SKILL.md` must follow progressive disclosure: keep the decision-critical rules an agent needs to execute the skill in the main file, and reserve support files for expanded examples, templates, references, scripts, or assets [because agents read skills under limited attention, so the first file must expose the decision-critical principles and constraints without forcing a deep reference chase]. See `examples/progressive-disclosure-examples.md` when the inline-vs-support boundary is unclear, the main file is becoming long or repetitive, or a support-file pointer lacks a read condition; do not see it for edits unrelated to progressive disclosure.
- **Remove inducements before adding counter-rules**: when an existing skill causes unwanted behavior because its wording, examples, or workflow make that behavior the obvious move, first name the inducing source and delete or neutralize it [because adding a counter-rule can keep the unwanted behavior salient and make the skill revolve around the behavior it should stop teaching]. Add a new constraint only when the behavior remains a real scope edge after the inducement is removed and the constraint can name an observable wrong-action condition.
- **Inline rationale rule**: not every principle or constraint needs a "why," but any rule that could be rationalized away or misapplied in a new scenario must include the causal mechanism. Required: frontmatter load conditions, non-obvious principles, and non-obvious constraints. Test: would an agent in a new scenario correctly apply this rule without knowing why it exists? If no → add inline rationale. Format: append the reason as a clause — "never add abstraction without a second caller [because premature abstraction creates coupling without evidence of need]."

## Constraints

When creating or optimizing a skill, keep these constraints true throughout the edit. Verify them from the edited skill or conversation evidence; if any constraint is violated, revise the skill.

- The frontmatter `description` must let an agent decide whether to load the skill before reading `SKILL.md` or any support files [because skill discovery happens from `name` and `description`; load conditions hidden in the body cannot help the pre-load decision].

- See `references/description-patterns.md` when creating or changing the frontmatter `description`; do not see it for body-only skill edits [because description rules are needed only for pre-load discovery wording, and unconditional reference-chasing wastes context].

- Constraints must be verifiable from observable evidence rather than advisory prose; principles must state action-shaping guidance rather than self-judgment prompts.

- Examples or support-file pointers must cover both positive use and boundary/failure cases.

- Do not name another skill as the next action instead of fixing that skill's own frontmatter `description` or local constraints.

- Do not add a new constraint, counter-rule, or `Do not use` rule to suppress behavior induced by existing wording, examples, or workflow. First name and remove or neutralize the inducing source; add a constraint only if the behavior remains a real scope boundary after that.

- Do not leave advisory prose where an observable condition or explicit red line is needed.

---

See `references/judgment-dimensions.md` when a skill creation or review needs deeper diagnosis than the main Principles and Constraints provide.
