---
name: using-skill-first
description: "Invoke once per session before the first action. Installs the habit of scanning the skill list before every subsequent step. DO NOT re-invoke mid-conversation — subsequent scans happen in thinking."
---

# Using Skills First

Check for applicable skills before every action — before each distinct step, including each item in a todo list.

## Trigger Logic

**Invocation default**: Invoke once at the start of each session, before the first action is taken. Once invoked, the discipline applies for the entire session. Do not re-invoke mid-conversation.

If the conversation begins with discussion or questions and no action is required, do not invoke yet — wait until the first action is about to be taken.

The cost of an unnecessary check is a brief lookup. The cost of skipping is taking action without guidance that could change the approach — and that guidance loss compounds across every subsequent step.

## Why This Matters

Skills encode judgment that took time to develop: what questions to ask before building, how to find root causes instead of symptoms, when to write the test before the code. When an agent skips the skill check, that accumulated experience is discarded. The agent reinvents reasoning the skill already provides — or worse, skips the reasoning entirely.

The most common failure is not "I checked and chose the wrong skill." It is "I started writing code / investigating / planning without checking at all."

## The 1% Rule

> If there is even a 1% chance a skill applies to what you are about to do, you must invoke it.

- This is non-negotiable
- This is not optional
- You cannot rationalize your way out of this

## Skill Checks Are Per-Action, Not Per-Conversation

Checking skills before the first action does not cover each subsequent action. Before executing any action — including each item in a todo list — check whether a skill applies to that specific action.

Listing todos is itself an action. Executing each todo is a separate action. Both require their own skill check.

## Skill Assessment Framework

Before any action, systematically assess applicable skills.

### For models with thinking mode:

**In your thinking process:**
1. **State the action** — What am I about to do? What is the goal?
2. **Go through the available skill list** — For each skill, read its description. Even a 1% match counts.
3. **For each potentially applicable skill, evaluate**: does the trigger condition match the current state? Am I certain it doesn't apply, or just assuming based on impression?
4. **Make decision**: invoke, skip (with stated observable reasoning), or uncertain (default: invoke)

**In your response:**
- If invoking skill(s) → proceed directly (skill invocation is visible to user)
- If skipping a skill that might seem applicable (>10% chance) → briefly state why in observable terms
- If uncertain → default to invoking

### For models without thinking mode:

Before any action, briefly assess:
1. What action am I taking?
2. Which skills might apply? Go through the skill list, check each description against what you're about to do. Apply the 1% rule.
3. For each: invoke or skip (state reasoning if skipping a seemingly applicable skill)

**Key principle**: A skill's description IS its trigger criterion. Read it against the current action — don't reason from memory about what the skill "is for."

When uncertain, default to invoking. The cost asymmetry favors false positives over false negatives.

## Capability Boundary

This skill establishes *when* to check for skills and *how* to assess them systematically. It does not tell you which specific skill to invoke in edge cases, rank skills by priority, or replace the guidance inside any individual skill.

## Rationalization Signals

These thoughts indicate a skill check is being skipped. Stop and check if you recognize any:

| Thought | What it signals |
|---------|-----------------|
| "I already checked skills before the first action" | That check covered the overall task, not this specific action |
| "This todo is small, no skill needed" | Small steps are where guidance is most often missed |
| "I'm in execution mode now, past the planning stage" | Execution steps can require their own skill checks |
| "I know what to do for this step" | Knowing what to do ≠ knowing the best way to do it |
| "I'll do this first, then check if needed" | The skill check must happen before the action, not after |
| "This is just a discussion, no task yet" | If an action is about to happen, this skill must be invoked first |
| "No skill in the list covers this exactly" | Read each description against the action — adjacent descriptions often apply |
| "I've used this pattern before, I don't need guidance" | Familiarity ≠ having the best approach; skills encode more than common patterns |

## Invariants

- Skill check precedes every action — not after, not during
- Listing a todo list does not substitute for checking skills before each todo
- The 1% rule has no exceptions for urgency, simplicity, or familiarity
- The check is against the available skill list, not memory of what skills exist

## Completion Criteria

- [ ] The actual skill list was scanned for the action about to be taken, not recalled from memory.
- [ ] For every skill with more than 1% applicability, it was either invoked or skipped with observable reasoning.
- [ ] The action is proceeding now rather than deferring skill checks to "when needed".

**If any criterion is not met, return to the Skill Assessment Framework before exiting.**

## Anti-Rationalization Check

Pause before proceeding to the action.

Do not treat this section as another checklist to clear. Use it to challenge whether the skill check that just happened was real.

Did I check the actual skill list, or did I rely on memory of which skills exist and what they cover?

Am I proceeding because the check was genuinely performed, or because checking now feels like unnecessary overhead given how obvious the next step seems?

## Failure Signals

Stop and reassess if:
- You are about to write code without having checked for relevant skills for that step
- You are executing a todo item without having checked for relevant skills for that item
- You last performed a skill check before the first action and have since taken multiple distinct actions without re-checking

