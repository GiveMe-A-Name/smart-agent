# Medium: Webhook Notifications

## Decision Brief

**Intent:** Let users receive job-completion results in their own systems without polling.

**Outcome:** A configured webhook receives the completed job result, transient delivery failures are retried, and jobs without a webhook keep their current completion behavior.

**Scope:** Job completion notifications, webhook configuration, and delivery resilience; no signing, multiple targets, or management UI.

**Visible Changes:**
- Webhook configured: deliver the job result to that URL.
- Temporary delivery failure: retry within a bounded policy.
- No webhook configured: complete the job without notification delivery.

**Approval Needed:** Support one webhook target with bounded in-process retry for this iteration; do not introduce multi-target or queue-backed delivery.

**Primary Risk:** The existing completion event may not contain enough data for the promised payload; prove that path before building configuration or retry behavior.

## Design Review

Job completion remains responsible only for lifecycle state and delegates optional delivery to notification delivery, which owns payload formatting, HTTP behavior, and bounded retry. Notification failure does not change the completed job state.

The material alternative is queue-backed delivery. It is excluded from this iteration because it adds durable infrastructure and a different reliability contract; if bounded in-process retry cannot meet the approved delivery behavior, stop rather than introducing a queue implicitly.

**Assessment**
- **Size:** Medium. Introduces a new notification concept and crosses job execution, configuration, HTTP behavior, and tests.
- **Nature:** Feature. No existing webhook system; config and retry decisions are needed.
- **Current state:** `job_runner.py` fires an `on_complete` event; config lives in `config.yaml`; no HTTP client is currently used.
- **Done state:** A completed job sends the expected payload to a configured URL, retries transient delivery failures within the approved bound, remains completed when delivery ultimately fails, and performs no delivery when no webhook is configured.

## Execution Plan

**Decomposition strategy**
- **Safe walking skeleton first:** connect job completion to one config-gated POST while preserving no-config behavior.
- **Risk-first ordering:** validate the `on_complete` event payload before building dependent behavior.
- **Vertical slicing:** every task adds verifiable behavior.

**Explicit exclusions**
- No webhook signing or authentication in this iteration.
- No multiple webhook targets per job.
- No webhook management UI; URL is config-only.

**Execution risks and plan-internal responses**
- **Risk:** Task 2's retry wiring may depend on the event payload and config-gated handoff proven in Task 1, making its current internal call shape premature. **Revision trigger:** refine that call shape after Task 1, provided the approved lifecycle/notification ownership and delivery behavior remain unchanged.

**Stop signals**
- If Task 1 confirms the Primary Risk, pause before designing a workaround.
- If bounded retry violates the existing completion-latency contract or requires a new external dependency, pause before changing the approved reliability approach.
- If implementation evidence requires changing the approved job-completion vs. notification-delivery responsibility split, pause before revising the plan.

**File map**
- `job_runner.py`: wire notification into completion handling.
- `config.yaml`: add webhook URL configuration.
- `notifications/webhook.py`: create the webhook sender.
- `notifications/retry.py`: add retry policy after the delivery path is proven.
- `tests/notifications/test_webhook.py`: cover payload, configuration, and retry behavior.
- `tests/test_job_runner.py`: prove notification failures do not change completed job state.

## Task 1: Config-Gated Walking Skeleton

Proves the completion event can produce one webhook payload end-to-end without changing behavior when no webhook is configured.

Files: `job_runner.py`; `config.yaml`; new `notifications/webhook.py`; `tests/notifications/test_webhook.py`; `tests/test_job_runner.py`.

- [ ] Add an optional webhook URL configuration with no configured default.
- [ ] Create `notifications/webhook.py` with `notify(job_result, webhook_url)` and call it from job completion only when the URL is present.
- [ ] Mock HTTP and assert a configured URL receives the expected job result payload.
- [ ] Add a no-config test proving no webhook is sent.
- [ ] Run `pytest tests/notifications/test_webhook.py tests/test_job_runner.py -v` -> PASS.
- [ ] Commit point: configured jobs send one webhook; unconfigured jobs preserve existing completion behavior.

## Task 2: Retry Behavior

Adds resilience after the event path and configuration are stable.

Depends on: Task 1's config-gated delivery path and payload contract.

Files: `notifications/webhook.py`; new `notifications/retry.py`; `tests/notifications/test_webhook.py`; `tests/test_job_runner.py`.

- [ ] Retry transient 5xx failures with backoff.
- [ ] Do not retry permanent 4xx failures.
- [ ] Add tests for retry, no-retry, success-after-retry, retry exhaustion, and network exceptions.
- [ ] Prove retry exhaustion and network exceptions leave the job in its completed state while recording final notification failure.
- [ ] Run `pytest tests/notifications/test_webhook.py tests/test_job_runner.py -v` -> PASS.
- [ ] Commit point: delivery failures are bounded and never rewrite completed job lifecycle state.

**Final verification:** Run `pytest tests/ -v && python scripts/smoke_webhook.py --url http://127.0.0.1:9000/webhook` -> PASS and prints `received expected webhook payload`.

## Execution Log

*(append entries here as each task completes - format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*
