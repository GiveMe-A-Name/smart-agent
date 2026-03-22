# Verification Notes

## Purpose

This file is a supporting appendix for verification.

The main job of this skill is still to help agents write better skills. Verification exists to check whether the guidance actually changed behavior.

## Evals First

For most non-trivial skills, the minimum useful verification is a small set of realistic eval prompts.

Good evals should target likely failure modes, distinguish a weak skill from a strong one, and check judgment rather than just formatting.

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
