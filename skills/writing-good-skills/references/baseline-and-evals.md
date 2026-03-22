# Verification Notes

## Purpose

This file is a supporting appendix for verification.

The main job of this skill is still to help agents write better skills. Verification exists to check whether the guidance actually changed behavior.

## Evals First

For most non-trivial skills, the minimum useful verification is a small set of realistic eval prompts.

Good evals should target likely failure modes, distinguish a weak skill from a strong one, and check judgment rather than just formatting.

They should also expose packaging mistakes, not just content mistakes.

Useful failure modes include:
- failing to notice repetition unless the user explicitly says the skill is too long
- moving core law out of the main file instead of only moving support material
- keeping heavy examples inline when the main file is already complete
- choosing the wrong support folder instead of following the canonical split guide in `references/lean-skill-patterns.md`

## When A Baseline Helps

You do not need a baseline for every small edit.

Use a baseline mainly when:
- you are heavily rewriting a non-trivial skill
- you are unsure what failure the skill is meant to correct
- the topic feels obvious and you want evidence instead of confidence

In that case, a baseline is just a quick before-state check: how does an agent respond without the improved skill?

Keep it small. Usually 2-3 prompts are enough.

## What To Record

If you run a baseline, record only:
- what the weak response did wrong
- what failure mode it revealed
- what the revised skill should improve

Do not turn this into a large testing workflow unless the skill truly needs it.

## When Evals Are Too Weak

If a weak draft still passes, do not keep polishing the prose.

Instead:
- strengthen the prompt
- make the expected failure mode more specific
- test a decision the weak draft is likely to miss

The eval should discriminate between a real skill and a polished note.

## Good Prompts For This Skill Family

Prompts are strong when they force the agent to choose between:
- concise law versus repeated explanation
- inline core guidance versus typed support folders
- reusable skill versus repo-local document

Example prompt shapes:
- "Review this vague skill. Do not rewrite the whole thing unless necessary."
- "This skill feels long but I did not say why. Improve it."
- "Should I move the trigger rules and core workflow into support files so `SKILL.md` stays short?"
- "Should these starter files be `templates/`, `examples/`, `references/`, or `scripts/`?"

## Expected Outcomes

Expected outcomes should be written in plain language.

Good:
- "Routes repo-local onboarding material to `AGENTS.md` or `README.md` instead of forcing a skill."
- "Rewrites the description around trigger conditions and explains why."

Weak:
- "Writes a good skill."
- "Improves the document."

## Minimal Heuristic

If your eval would still pass for a vague or generic skill, the eval is too weak.

If your baseline process becomes longer than the skill draft itself, you are probably overdoing it.

If the prompt only tests whether the agent can write nice prose, it is too weak for skill design.
