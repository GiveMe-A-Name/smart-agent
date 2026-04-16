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

---

## Before / After: What Good Looks Like

### Bad Layer 1 (technical language, code references)

> - **Goal:** Fix `auth_middleware.py:47` to return 401 instead of 404
> - **Why:** `token.exists()` is checked before `token.is_valid()`, causing 404 on the exists path
> - **Expected outcome:** `authenticate()` returns `HTTPException(status_code=401)` for expired tokens
> - **Impact scope:** `auth_middleware.py`, `tests/test_auth.py`
> - **Size:** Tiny

**Problems:** Goal names a file and line number. Why describes internal call order. Expected outcome cites a function signature. Impact scope lists file paths. A PM reads this and learns nothing about what was broken or what gets fixed.

### Good Layer 1 (domain language, outcome-focused)

> - **Goal:** Fix expired tokens returning the wrong error code
> - **Why:** Users see "not found" errors when their session expires, making them think the feature is broken rather than their login
> - **Expected outcome:** Expired tokens now correctly return "Unauthorized" so users know to log in again
> - **Impact scope:** Authentication middleware only
> - **Size:** Tiny

**Why this works:** A PM can read it, understand what was wrong, and judge whether the fix is the right call — without knowing anything about the codebase.

---

### Bad Layer 2 (implementation-level, not strategic)

> 1. Modify `notifications/webhook.py` to add `retry()` method
> 2. Update `job_runner.py:on_complete` to call `WebhookNotifier.notify()`
> 3. Add `webhooks.url` key to `config.yaml` schema

**Problem:** This describes what code changes, not *why* the plan is shaped this way. A reviewer can't tell whether the ordering is intentional or whether the approach is sound.

### Good Layer 2 (strategy and reasoning)

> Build incrementally, tackling the riskiest part first — wiring into the job completion event. Three steps:
> 1. Get a hardcoded webhook firing end-to-end (proves the approach works before investing in config or retry)
> 2. Make the URL configurable (low risk, straightforward)
> 3. Add retry on failure (well-understood pattern)
>
> Main risk: the job completion event might not carry enough context for a useful payload. We find out in step 1, before committing to the rest.

**Why this works:** Explains the *strategy* and *why the order matters*. A reviewer can now judge whether the approach is sound, not just whether the steps are listed.
