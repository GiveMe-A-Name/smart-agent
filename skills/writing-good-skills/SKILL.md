---
name: writing-good-skills
description: "Create, refine, or audit agent skills so they stay compact, discoverable, and high-signal. Use when writing a new skill, tightening or reviewing an existing SKILL.md, or improving a skill's description and structure."
---

# Writing Good Skills

Write the smallest skill that reliably fixes a real agent failure. Omit generic guidance the model already follows.

## Ground In Real Gaps

- Start from concrete evidence—repeated corrections, failed runs, non-obvious project conventions, or domain gotchas—not from an important-sounding topic.
- Keep plausible but unobserved edge cases out; do not turn a narrow failure mode into a comprehensive handbook.
- When refining a skill, remove wording or examples that induce unwanted behavior before adding counter-rules. Delete any instruction whose removal does not degrade outcomes.

## Match Control To Fragility

- Choose the lightest control that holds: principles and tradeoffs for context-dependent judgment, a default plus escape condition when one path usually wins, checklists or gates when omissions are the failure mode, and scripts when deterministic logic would otherwise be reimplemented.
- Prefer one working default over a menu of equivalent options. Do not freeze flexible judgment into rigid headings or templates.

## Structure For Discovery And Loading

- **Description:** State the capability plus concrete, pre-load trigger signals (requests, artifacts, or task states visible before opening `SKILL.md`). Keep workflow steps and post-investigation conditions out of the description.
- **Progressive disclosure:** Keep always-needed rules and unrecognized gotchas in `SKILL.md`—if an agent cannot tell a gotcha exists, it will never open a sub-file to find it. Put conditional detail in focused `references/`, templates in `assets/`, and deterministic helpers in `scripts/`, giving every pointer an observable read condition.
- **Verify impact:** Check the skill on realistic prompts, including adjacent near misses that should not trigger. Cut or retire guidance that adds context cost without improving behavior.
