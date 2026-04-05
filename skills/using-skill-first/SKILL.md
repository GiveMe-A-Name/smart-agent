---
name: using-skill-first
description: "Invoke once per session before the first action. Installs the habit of scanning the skill list before every subsequent step. DO NOT re-invoke mid-conversation — subsequent scans happen in thinking."
---

# Using Skills First

Check for applicable skills before every action — before each distinct step, including each item in a todo list.

The most common failure is not "I checked and chose the wrong skill." It is "I started acting without checking at all."

## Trigger Logic

Invoke once per session, before the first action. Once invoked, the discipline applies for the entire session — do not re-invoke.

If the conversation begins with discussion and no action is required, wait until the first action is about to be taken.

## What Counts as an Action

An action is any step that changes state or commits to a direction. The scan obligation applies per action.

**Is an action:**
- Tool call (read, write, edit, bash, grep, glob, etc.)
- Creating or updating a plan / todo list
- Invoking a skill
- Deciding on an implementation approach and starting to execute it

**Not an action:**
- Reasoning or analysis in thinking (no external effect)
- Asking the user a clarifying question
- Summarizing or restating what the user said

When ambiguous, treat it as an action. The cost of one extra scan is negligible.

## The Scan-Then-Decide Rule

Before every action, **scan** the skill list. The scan has two tiers:

### Tier 1: Full Scan (session start + context shift)

Scan every skill description against the current action. Do this:
- Before the **first action** of the session
- When the **task nature changes** (e.g., writing code → debugging, coding → reviewing a PR)

After a full scan, mark a **working set** — the subset of skills plausibly relevant to the current task context.

### Tier 2: Working-Set Scan (subsequent actions)

Before each subsequent action within the same task context:
1. Scan the working set against this specific action.
2. Quick-check: has the task nature shifted enough to warrant a full scan? If yes, do Tier 1 instead.

This keeps scan cost manageable without skipping the check entirely.

### Scan Output Format

Each scan must produce structured output in thinking, not free-form reasoning:

```
[skill-scan] action: <what I am about to do>
[skill-scan] tier: full | working-set
[skill-scan] candidates:
  - <skill-name>(invoke: <reason>)
  - <skill-name>(skip: <specific reason why it does not apply to THIS action>)
[skill-scan] no candidates — proceeding
```

A skip reason of "not applicable" or "not relevant" is not specific enough. Name what about this action makes the skill not apply.

### Invoke vs. Skip

- **Invoke** — the skill plausibly applies. Call it.
- **Skip with reason** — the skill looked relevant on scan but does not actually apply to this specific action. State why in one concrete sentence.

When uncertain between invoke and skip, default to invoke. The cost of an unnecessary invocation is one skill pass. The cost of a wrong skip is acting without guidance that could change the approach.

## Skill Checks Are Per-Action, Not Per-Conversation

Checking before the first action does not cover subsequent actions. Before executing any action — including each item in a todo list — check whether a skill applies to that specific action.

Listing todos is itself an action. Executing each todo is a separate action. Both require their own skill check.

## Skill Assessment

Before any action:
1. **State the action** — What am I about to do?
2. **Scan** — Tier 1 or Tier 2 as appropriate. Use the structured output format.
3. **Decide per candidate** — Invoke, or skip with a concrete reason. When uncertain, invoke.

## Anti-Pattern: What Goes Wrong Without Scanning

A user asks the agent to fix a failing test. The agent reads the error, identifies the fix (a wrong return type), and edits the file. No skill scan.

What was missed: `systematic-debugging` would have prompted root-cause investigation instead of symptom patching. The return type was wrong because an upstream function changed its contract — the "fix" passed the test but introduced a silent data corruption three calls downstream.

The agent knew what to do. It did not know the best way. The skill scan would have caught that.

## Rationalization Signals

If you catch yourself thinking any of these, stop and check:

| Thought | What it signals |
|---------|-----------------|
| "I already checked skills at the start" | That check covered the first action, not this one |
| "This step is small, no skill needed" | Small steps are where guidance is most often missed |
| "I know what to do here" | Knowing what to do ≠ knowing the best way |
| "I'll do this first, then check if needed" | The check must happen before the action, not after |
| "No skill covers this exactly" | Read each description against the action — adjacent skills often apply |
| "I scanned and nothing matched" | If no candidate survived scan, fine. If candidates were dismissed without a specific reason, the scan was incomplete |

## Invariants

- Skill scan precedes every action — not after, not during
- Listing a todo list does not substitute for scanning skills before each todo
- The scan threshold has no exceptions for urgency, simplicity, or familiarity
- Skipping a candidate skill without a concrete, action-specific reason is a rule violation
- The scan is against the available skill list, not memory of what skills exist
- A skip reason must name what about this action makes the skill not apply — generic reasons ("not relevant") are insufficient
- A context shift (task nature change) requires a Tier 1 full scan, not continued working-set scanning
