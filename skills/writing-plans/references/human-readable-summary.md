# Human Review Section

Plan documents serve two audiences: **agents** (who execute) and **humans** (who approve intent, scope, and strategy). These audiences need different things from the same document.

The Human Review Section is the human's surface. It is written once, approved once, and then **frozen for the life of the plan**. Agents read it for context but never modify it. This freeze is what makes review meaningful — a human can trust that what they approved is still what the agent is executing against.

The execution detail (task checklists, file paths, verification commands) lives below the Human Review Section. Agents work there. Humans may read it, but they are not required to.

---

## Intent Statement (1-second read)

The very first line of the Human Review Section. One sentence stating the user's requested outcome — framed from the user's perspective, not the agent's task list.

> **Intent:** [One sentence — what the user wants to achieve, in their terms]

This is an alignment checkpoint. The user reads this line first and either confirms the direction or corrects it before reviewing anything else. If this line is wrong, the rest of the plan doesn't matter.

**How to write it:** Mirror the user's goal, not the implementation. "Move all logging and metrics dependencies into the shared package so individual Next.js and Express apps no longer bundle them separately" — not "Modify `apps/web/package.json` and `apps/api/package.json` to remove winston and prom-client imports."

---

## Layer 1: Overview (5-second read)

A short block that lets a human decide "direction correct or not" at a glance:

- **Goal** — one sentence: what are we doing
- **Why** — one sentence: what problem this solves or what value it adds
- **Expected outcome** — what the system looks like after execution (user-visible or developer-visible changes, not file paths)
- **Impact scope** — which modules/features/domains are affected (human concepts, not file paths)
- **Size** — Tiny / Small / Medium / Large

## Layer 2: Approach Overview (30-second read)

A short paragraph or bullet list that describes **how** in human terms — the strategy, key steps, and important design decisions. Use domain concepts ("add retry logic to webhook sender"), not code-level references ("modify `notifications/webhook.py:send()`"). Include trade-offs or alternatives considered if they affect scope, user-visible behavior, risk, or dependencies.

Layer 2 passes when it states ordering logic and major trade-offs without file paths or task checklists. If it only lists code edits, rewrite it as strategy: what happens first, why that order reduces risk, and what decision the human is approving.

## Task Overview (medium/large only)

One sentence per task, in plain English, explaining what each task achieves and why it comes at this point in the sequence. This lets the human verify the ordering logic without reading the execution detail.

Example:
> - Task 1: Prove the approach works end-to-end before investing in configuration or retry logic
> - Task 2: Make the webhook URL configurable — low risk, builds on the proven skeleton
> - Task 3: Add retry on failure — comes last because the assessment found existing retry test patterns to copy and it depends on the working base

No file paths. No method names. If a task's purpose can't be stated in one human sentence, the task needs rethinking.

## Key Decisions (medium/large only)

Each design choice that requires human ratification. Format:

> **[Decision]:** [alternatives considered] → [chosen approach and why]

Only include decisions where a reasonable person might have chosen differently. Obvious choices with no real alternative can be omitted.

Example:
> **Retry strategy:** client-side retry with exponential backoff vs. queue-based retry → chose client-side because job completion is already synchronous and introducing a queue would require new infrastructure for a non-critical feature.

---

## Scaling Rules

- **All sizes** — Intent Statement always required. It is the first line of the Human Review Section regardless of plan size.
- **Tiny/Small** — Intent Statement + Layer 1 only. The plan itself is short enough that a human can read it directly.
- **Medium/Large** — Intent Statement + Layer 1 + Layer 2 + Task Overview + Key Decisions. The technical detail is dense enough that humans need a complete separate entry point.

## Freeze Rules

The Human Review Section is frozen at approval. Mark it with `[APPROVED — READ ONLY]` when the human confirms.

- **During execution**: read for context, never modify
- **During plan revision**: if not-yet-started tasks change, update the Execution Log — not the Human Review Section
- **If the contract breaks**: if execution reveals the goal or approach in the Human Review Section is fundamentally wrong, stop and ask the human to re-approve a new plan. Do not silently update the frozen section.

## Writing Principles

- Use **human concepts**, not code concepts — "notification system" not "`notifications/webhook.py`"
- Use **outcome language**, not implementation language — "users receive webhook calls when jobs complete" not "POST request is sent in `on_complete` handler"
- Keep it bounded — Layer 1 has exactly the five fields listed above; Layer 2 is either 3-5 bullets or one paragraph under 120 words
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

**Why this works:** It names the user-visible error, the corrected behavior, and the affected product area without file paths, function names, or implementation mechanics.

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
> 3. Add retry on failure (existing retry test patterns are available to copy)
>
> Main risk: the job completion event might not carry enough context for a useful payload. We find out in step 1, before committing to the rest.

**Why this works:** Explains the *strategy* and *why the order matters*. A reviewer can now judge whether the approach is sound, not just whether the steps are listed.
