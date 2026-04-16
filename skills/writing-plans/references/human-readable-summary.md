# Human-Readable Summary

Plan documents serve two audiences: **agents** (who execute) and **humans** (who approve and understand). The technical task details are agent-friendly but hard for humans to scan. The summary provides progressive disclosure — humans read as deep as they need to judge the plan.

---

## Layer 1: Overview (5-second read)

A short block that lets a human decide "direction correct or not" at a glance:

- **Goal** — one sentence: what are we doing
- **Why** — one sentence: what problem this solves or what value it adds
- **Expected outcome** — what the system looks like after execution (user-visible or developer-visible changes, not file paths)
- **Impact scope** — which modules/features/domains are affected (human concepts, not file paths)
- **Size** — Tiny / Small / Medium / Large

## Layer 2: Approach Overview (30-second read)

A short paragraph or bullet list that describes **how** in human terms — the strategy, key steps, and important design decisions. Use domain concepts ("add retry logic to webhook sender"), not code-level references ("modify `notifications/webhook.py:send()`"). Include trade-offs or alternatives considered if they matter for approval.

If the human reads Layer 1 and still can't judge whether the direction is correct, Layer 2 gives enough strategic detail to make that call without reading file paths or task checklists.

---

## Scaling Rules

- **Tiny/Small** — Layer 1 only. The plan itself is short enough that a human can read it directly. Layer 2 would repeat the plan.
- **Medium/Large** — Both layers required. The technical detail is dense enough that humans need a separate readable entry point.

## Update Rules

When a plan revision occurs during execution, update the summary to reflect the revised direction. A stale summary is worse than no summary — it misleads the human reviewer. Layer 1 and Layer 2 must stay consistent with the current plan state.

## Writing Principles

- Use **human concepts**, not code concepts — "notification system" not "`notifications/webhook.py`"
- Use **outcome language**, not implementation language — "users receive webhook calls when jobs complete" not "POST request is sent in `on_complete` handler"
- Keep it **short** — Layer 1 should be 5 lines; Layer 2 should be 3-5 bullet points or a short paragraph
- **No file paths** in the summary — that's what the technical plan is for
