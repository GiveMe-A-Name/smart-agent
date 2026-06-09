# Medium: Webhook Notifications

**Situation:** When a job completes, the system should POST the job result to a user-configured webhook URL.

**Assessment**
- **Size:** Medium. Introduces a new notification concept and crosses job execution, configuration, HTTP behavior, and tests.
- **Nature:** Feature. No existing webhook system; config and retry decisions are needed.
- **Current state:** `job_runner.py` fires an `on_complete` event; config lives in `config.yaml`; no HTTP client is currently used.
- **Done state:** A completed job sends the expected payload to a configured URL, retries on transient failure, and tests pass.

## Human Review Section

> **Intent:** Let users receive real-time job-completion notifications in their own systems without polling.
>
> **Summary (Layer 1)**
> - **Goal:** Send webhook notifications when jobs complete.
> - **Why:** Users need job results delivered to external systems automatically.
> - **End state:** A completed job delivers its result to a configured webhook URL and retries transient failures.
> - **Impact scope:** Job completion behavior, configuration, and notification delivery.
> - **Size:** Medium.
>
> **Summary (Layer 2 - Approach)**
> Build a walking skeleton first: send one hardcoded webhook from the job-completion event before adding configuration and retry behavior. This resolves the main uncertainty early: whether the completion event carries enough data for a useful payload.
>
> **Change Snapshot**
> - Job completes with webhook URL: job result is delivered to that URL.
> - Webhook delivery fails temporarily: delivery is retried according to the retry policy.
> - No webhook URL configured: job completion still works without notification delivery.
>
> **Task Overview**
> - **Task 1:** Prove one webhook can fire from job completion before investing in configuration or retries.
> - **Task 2:** Move the URL into configuration after the event path is proven.
> - **Task 3:** Add retry behavior last, once the sender and config path are stable.
>
> **Key Decisions**
> - **Delivery scope:** Single webhook target vs. multiple targets -> choose one target for this iteration to keep behavior reviewable.
> - **Retry scope:** Simple client-side retry vs. queue-backed delivery -> choose client-side retry because webhook delivery is useful but not critical job infrastructure.
>
> **Conflict Priority**
> When completeness conflicts with a working subset, prefer the working subset. A configurable webhook is more important than retry behavior.

## Execution Plan

**Decomposition strategy**
- **Walking skeleton first:** connect job completion to one POST before adding config or retry.
- **Risk-first ordering:** validate the `on_complete` event payload before building dependent behavior.
- **Vertical slicing:** every task adds verifiable behavior.

**Explicit exclusions**
- No webhook signing or authentication in this iteration.
- No multiple webhook targets per job.
- No webhook management UI; URL is config-only.

**Stop signals**
- If `on_complete` does not provide enough data for a useful payload, pause before designing a workaround.
- If retry requires a new external dependency, pause before adding it.

**File map**
- `job_runner.py`: wire notification into completion handling.
- `config.yaml`: add webhook URL configuration.
- `notifications/`: create webhook sender module.
- `tests/notifications/`: add notification tests.

## Task 1: Walking Skeleton

Proves the completion event can produce one webhook payload end-to-end.

Files: `job_runner.py`; new webhook sender module in `notifications/`; tests in `tests/notifications/`.

- [ ] Create a sender with `notify(job_result)` that POSTs to a hardcoded URL.
- [ ] Mock HTTP and assert the POST payload contains the job result fields.
- [ ] Call the sender from the job-completion path.
- [ ] Run `pytest tests/notifications/ -v` -> PASS.
- [ ] Commit point: a completed job triggers one hardcoded webhook.

## Task 2: Configuration

Makes the proven webhook path usable without code changes.

Files: `config.yaml`; webhook sender from Task 1; `job_runner.py`.

- [ ] Add webhook URL configuration.
- [ ] Read the URL from config instead of using a hardcoded value.
- [ ] Skip webhook creation when the URL is absent.
- [ ] Add a no-config test proving no webhook is sent.
- [ ] Run `pytest tests/ -v` -> PASS.
- [ ] Commit point: webhook delivery is config-driven.

## Task 3: Retry Behavior

Adds resilience after the event path and configuration are stable.

Files: webhook sender and its tests.

- [ ] Retry transient 5xx failures with backoff.
- [ ] Do not retry permanent 4xx failures.
- [ ] Add tests for retry, no-retry, and success-after-retry.
- [ ] Run `pytest tests/notifications/ -v` -> PASS.
- [ ] Commit point: retry behavior is complete.

**Final verification:** Run a test job with the webhook URL pointed at a local echo server and confirm the expected payload is received.
