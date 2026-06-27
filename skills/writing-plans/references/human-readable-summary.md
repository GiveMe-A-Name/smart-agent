# Human Review Section

Plan documents serve two audiences: **agents** (who execute) and **humans** (who approve intent, scope, rationale, and solution shape). These audiences need different things from the same document.

The Human Review Section is the human's approval-understanding surface. It is written once, approved once, and then **frozen for the life of the plan**. Agents read it for context but never modify it. This freeze is what makes review meaningful — a human can trust that what they approved is still what the agent is executing against.

The execution detail (task checklists, file paths, verification commands) lives below the Human Review Section. Agents work there. Humans may read it, but they are not required to.

The section must let the human answer these alignment questions before execution starts:
- Is this the problem I meant?
- Is this the outcome and impact scope I want?
- Do I understand the strategy and why this plan uses it?
- Do I understand what shape the code or system will have after this plan?
- Are the important trade-offs, excluded alternatives, and conflict priorities visible?
- Do I know what the agent should preserve if implementation gets messy?

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
- **End state** — one concise sentence: what will be true after execution in observable user, developer, or domain terms
- **Impact scope** — which modules/features/domains are affected, plus approval-critical non-areas when adjacent behavior could plausibly change (human concepts, not file paths)
- **Size** — Tiny / Small / Medium / Large

End state is the human's completion anchor. It must not list implementation steps, task names, file paths, commands, internal APIs, or implementation-only function/method/class/module names. If the final state needs multiple concrete deltas, keep End state high-level and use Change Snapshot when triggered.

## Layer 2: Approach And Design Rationale (30-second read)

A short paragraph or bullet list that describes **how and why** in human terms: the strategy, the design intent, the property the plan is protecting, the ordering logic, and important trade-offs. Use domain concepts ("add retry logic to webhook sender"), not code-level references ("modify `notifications/webhook.py:send()`"). Include alternatives considered when they affect scope, user-visible behavior, risk, dependencies, responsibility ownership, or long-term code shape.

Layer 2 passes when it states approach rationale and ordering logic without file paths or task checklists. If it only lists code edits, rewrite it as design reasoning: what strategy the plan uses, why it fits current evidence, what should be preserved, what happens first, and why that order reduces risk.

## Change Snapshot (medium/large only, conditional)

Add Change Snapshot for medium/large plans when any trigger below appears. Omit the field for tiny/small plans and for medium/large plans without these triggers.

Use Change Snapshot when the intent, assessment, risks, Key Decisions, stop signals, or task outcomes include any of these observable triggers:
- Public/developer-facing contract changes
- Compatibility expectations or behavior that must remain unchanged
- Data or contract migrations
- Restored old behavior
- Multiple input/output outcomes
- Multiple valid interpretations of the target behavior

Choose the shortest readable form for the contract:
- Scenario bullets when behavior depends on runtime conditions
- Before/After pairs when the change is easiest to compare directly
- Public API/schema/CLI diffs when developer-facing contracts are the review surface
- A small behavior matrix when multiple inputs map to different outcomes
- Invariants when the main risk is preserving existing behavior

Change Snapshot contains the minimum complete visible delta needed for approval. Keep it to one compact block when possible. If the complete approval delta is too large for one compact block, compress dimensions, move implementation detail to the technical plan, surface ratification choices in Key Decisions, or stop for human scoping; do not move approval-critical contract detail out of the Human Review Section. Change Snapshot answers what visible behavior changes; Layer 2 and Concrete Design Sketch answer why this approach and shape are being used.

Do not include file paths, task references, implementation-only names, or explanatory prose. Public/developer-facing endpoints, fields, schema names, config keys, event names, CLI options, public SDK/plugin methods, and contract names are allowed only when they are the behavior being approved.

Scenario example:
> **Change Snapshot**
> - Job completes with webhook URL: result is delivered to that URL.
> - Webhook delivery fails temporarily: delivery is retried according to the retry policy.
> - No webhook URL configured: current job completion behavior stays unchanged.

Public API diff example:
> **Change Snapshot**
> | Contract | Before | After |
> | --- | --- | --- |
> | `POST /jobs` response | returns job id only | returns job id and webhook delivery status |
> | `webhook_url` | unsupported | optional field; omitted keeps current behavior |

## Task Overview (medium/large only)

One sentence per task, in plain English, explaining what each task achieves and why it comes at this point in the sequence. This lets the human verify the ordering logic and risk-reduction strategy without reading the execution detail.

Example:
> - Task 1: Prove the approach works end-to-end before investing in configuration or retry logic
> - Task 2: Make the webhook URL configurable — low risk, builds on the proven skeleton
> - Task 3: Add retry on failure — comes last because the assessment found existing retry test patterns to copy and it depends on the working base

No file paths. No method names. If a task's purpose can't be stated in one human sentence, the task needs rethinking.

## Key Decisions (medium/large only)

Each design choice that requires human ratification. Include choices about responsibility ownership, compatibility, abstraction, dependency, rollout, scope, and excluded alternatives when a reasonable human could choose differently. Format:

> **[Decision]:** [alternatives considered] → [chosen approach and why]

Only include decisions where a reasonable person might have chosen differently. Obvious choices with no real alternative can be omitted. If a medium/large plan has no decisions requiring human ratification, write `None requiring human ratification` so the absence is explicit.

Example:
> **Retry strategy:** client-side retry with exponential backoff vs. queue-based retry → chose client-side because job completion is already synchronous and introducing a queue would require new infrastructure for a non-critical feature.

Ownership example:
> **Notification ownership:** keep delivery logic inside job completion vs. move it into notification delivery -> chose notification delivery so job completion remains responsible only for lifecycle state.

## Conflict Priority (medium/large when goals can compete)

When two plan goals can conflict, state the tiebreaker in the Human Review Section:

> **Conflict priority:** When [X] conflicts with [Y], prefer [X] because [reason].

Omit this field only when no competing goals are named or implied by the plan.

## Concrete Design Sketch (conditional)

When the Concrete Design Sketch Gate is triggered, place `### Concrete Design Sketch` inside the Human Review Section, after the summary/decision fields and before `## Situation Assessment`. This placement is intentional: the sketch belongs to approval understanding, not to the execution checklist.

The sketch answers: "what solution shape and code architecture am I approving?" It uses reader-facing architecture, responsibility, boundary, and flow language:
- **Design intent:** what this structure is protecting
- **Target code architecture:** the post-change topology: main packages/modules/layers, entrypoint roles, shared seams, dependency direction, and ownership boundaries. This is an architecture map, not a file map.
- **Resulting responsibility shape:** which responsibility blocks exist after the change
- **Main flow:** how the user action, event, data, or control path moves through those blocks
- **Boundaries changed or preserved:** what owns state, validation, persistence, side effects, orchestration, or presentation
- **Misalignment shape:** what implementation shape would satisfy behavior while showing the design was misunderstood

Do not use task references, line numbers, implementation-only function names, private interface details, or exact edit lists in the sketch. Stable package names, public contract names, and directory-level architecture anchors are allowed when they are needed to show the target code architecture. Exact files-to-edit belong in the File Map, not the Human Review Section.

---

## Scaling Rules

- **All sizes** — Intent Statement always required. It is the first line of the Human Review Section regardless of plan size.
- **Tiny/Small** — Intent Statement + Layer 1 only. The plan itself is short enough that a human can read it directly.
- **Medium/Large** — Intent Statement + Layer 1 + Layer 2 Approach and Design Rationale + Change Snapshot when triggered + Task Overview + Key Decisions (or `None requiring human ratification`) + Conflict Priority when goals can compete + Concrete Design Sketch when triggered. The technical detail is dense enough that humans need a complete separate entry point.

## Freeze Rules

The Human Review Section is frozen at approval. Mark it with `[APPROVED — READ ONLY]` when the human confirms.

- **During execution**: read for context, never modify
- **During plan revision**: if not-yet-started tasks change, update the Execution Log — not the Human Review Section
- **If the contract breaks**: if execution reveals the goal or approach in the Human Review Section is fundamentally wrong, stop and ask the human to re-approve a new plan. Do not silently update the frozen section.

## Writing Principles

- Use **human concepts**, not code concepts — "notification system" not "`notifications/webhook.py`"
- Use **outcome language**, not implementation language — "users receive webhook calls when jobs complete" not "POST request is sent in `on_complete` handler"
- Use **rationale language**, not task-list language, in Layer 2 — "prove the event path before adding retry because payload availability is the main uncertainty" not "edit sender, then config, then retry"
- Keep it bounded — Layer 1 has exactly the five fields listed above; Layer 2 is either 3-5 bullets or one paragraph under 140 words; Change Snapshot contains only the minimum complete visible delta needed for approval
- If a field exceeds its limit, revise by deleting lower-value detail or switching to a more compact snapshot format instead of adding explanatory prose
- **No exact files-to-edit** in the Human Review Section — that's what the technical plan is for. Stable package names and directory-level architecture anchors are allowed in Concrete Design Sketch when they are needed to show the approved code architecture.

---

## Before / After: What Good Looks Like

### Bad Layer 1 (technical language, code references)

> - **Goal:** Fix `auth_middleware.py:47` to return 401 instead of 404
> - **Why:** `token.exists()` is checked before `token.is_valid()`, causing 404 on the exists path
> - **End state:** `authenticate()` returns `HTTPException(status_code=401)` for expired tokens
> - **Impact scope:** `auth_middleware.py`, `tests/test_auth.py`
> - **Size:** Tiny

**Problems:** Goal names a file and line number. Why describes internal call order. End state cites a function signature. Impact scope lists file paths. A PM reads this and learns nothing about what was broken or what gets fixed.

### Good Layer 1 (domain language, outcome-focused)

> - **Goal:** Fix expired tokens returning the wrong error code
> - **Why:** Users see "not found" errors when their session expires, making them think the feature is broken rather than their login
> - **End state:** Expired tokens now correctly return "Unauthorized" so users know to log in again
> - **Impact scope:** Authentication middleware only
> - **Size:** Tiny

**Why this works:** It names the user-visible error, the corrected behavior, and the affected product area without file paths, function names, or implementation mechanics.

---

### Bad Layer 2 (implementation-level, not rationale)

> 1. Modify `notifications/webhook.py` to add `retry()` method
> 2. Update `job_runner.py:on_complete` to call `WebhookNotifier.notify()`
> 3. Add `webhooks.url` key to `config.yaml` schema

**Problem:** This describes what code changes, not *why* the plan is shaped this way. A reviewer can't tell whether the ordering is intentional, what the design is protecting, or whether the approach is sound.

### Good Layer 2 (approach and design rationale)

> Build incrementally, tackling the riskiest part first — wiring into the job completion event. Three steps:
> 1. Get a hardcoded webhook firing end-to-end (proves the approach works before investing in config or retry)
> 2. Make the URL configurable (low risk, straightforward)
> 3. Add retry on failure (existing retry test patterns are available to copy)
>
> Design intent: keep job completion as the lifecycle boundary and move delivery policy into notification delivery. Main risk: the job completion event might not carry enough context for a useful payload. We find out in step 1, before committing to the rest.

**Why this works:** Explains the strategy, design intent, and why the order matters. A reviewer can now judge whether the approach matches the intended code shape, not just whether the steps are listed.
