---
name: use-skill-first
description: Use at the start of any conversation — establishes the discipline of checking for applicable skills before any action, including before executing each individual todo item.
---

# Use Skill First

Check for applicable skills before every action — not just at conversation start, but before each distinct step, including each item in a todo list.

## Trigger Logic

**Invocation default**: Invoke at the start of every conversation. The cost of an unnecessary invocation is a brief check. The cost of skipping is taking action without available guidance that could change the approach.

Do not invoke again mid-conversation if a skill check was already performed for the current action.

## The 1% Rule

> If there is even a 1% chance a skill applies to what you are about to do, you must invoke it.

- This is non-negotiable
- This is not optional
- You cannot rationalize your way out of this

## Skill Checks Are Per-Action, Not Per-Conversation

Checking skills at the start of a conversation does not cover each subsequent action. Before executing any action — including each item in a todo list — check whether a skill applies to that specific action.

Listing todos is itself an action. Executing each todo is a separate action. Both require their own skill check.

## Capability Boundary

This skill establishes *when* to check for skills. It does not tell you which skill to invoke, rank skills by priority, or replace the guidance inside any individual skill.

## Rationalization Signals

These thoughts indicate a skill check is being skipped. Stop and check if you recognize any:

| Thought | What it signals |
|---------|-----------------|
| "I already checked skills at the start" | That check covered the overall task, not this specific action |
| "This todo is small, no skill needed" | Small steps are where guidance is most often missed |
| "I'm in execution mode now, past the planning stage" | Execution steps can require their own skill checks |
| "I know what to do for this step" | Knowing what to do ≠ knowing the best way to do it |
| "I'll do this first, then check if needed" | The skill check must happen before the action, not after |

## Invariants

- Skill check precedes every action — not after, not during
- Listing a todo list does not substitute for checking skills before each todo
- The 1% rule has no exceptions for urgency, simplicity, or familiarity

## Failure Signals

Stop and reassess if:
- You are about to write code without having checked for relevant skills for that step
- You are executing a todo item without having checked for relevant skills for that item
- You last performed a skill check at conversation start and have since taken multiple distinct actions without re-checking
