---
name: using-skill-first
description: Use once per session, before the first action is taken, to establish the habit of checking for applicable skills before every action.
---

# Using Skill First

Check for applicable skills before every action — before each distinct step, including each item in a todo list.

## Trigger Logic

**Invocation default**: Invoke once per session, before the first action is taken. Once invoked, the discipline applies for the entire session. Do not re-invoke mid-conversation.

If the conversation begins with discussion or questions and no action is required, do not invoke this skill yet — wait until the first action is about to be taken.

The cost of an unnecessary invocation is a brief check. The cost of skipping is taking action without available guidance that could change the approach.

## The 1% Rule

> If there is even a 1% chance a skill applies to what you are about to do, you must invoke it.

- This is non-negotiable
- This is not optional
- You cannot rationalize your way out of this

## Skill Checks Are Per-Action, Not Per-Conversation

Checking skills before the first action does not cover each subsequent action. Before executing any action — including each item in a todo list — check whether a skill applies to that specific action.

Listing todos is itself an action. Executing each todo is a separate action. Both require their own skill check.

## Skill Assessment Framework

Before any action, systematically assess applicable skills using this framework.

### For models with thinking mode:

**In your thinking process:**
1. **State the action** - What am I about to do? What is the goal?
2. **List potentially applicable skills** - Even 1% chance counts. Go through the available skill list.
3. **For each potentially applicable skill, evaluate**:
   - Does the trigger condition match the current state?
   - Am I certain it doesn't apply, or just assuming based on impression?
   - What's the cost of invoking vs missing?
4. **Make decision**: invoke, skip (with observable reasoning), or uncertain (default to invoke)

**In your response:**
- If invoking skill(s) → proceed directly (skill invocation is visible to user)
- If skipping a skill that might seem applicable (>10% chance) → briefly state why in observable terms
- If uncertain → output the assessment and ask for guidance

### For models without thinking mode:

Before any action, briefly assess:
1. What action am I taking?
2. Which skills might apply (1% rule)?
3. For each: invoke or skip (state reasoning if skipping seemingly applicable skills)

**Key principle**: When uncertain, default to invoking. The cost asymmetry favors false positives over false negatives.

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

## Invariants

- Skill check precedes every action — not after, not during
- Listing a todo list does not substitute for checking skills before each todo
- The 1% rule has no exceptions for urgency, simplicity, or familiarity

## Failure Signals

Stop and reassess if:
- You are about to write code without having checked for relevant skills for that step
- You are executing a todo item without having checked for relevant skills for that item
- You last performed a skill check before the first action and have since taken multiple distinct actions without re-checking
