# Anti-Pattern: Horizontal Slicing

The webhook feature planned as layers instead of vertical behavior:

```markdown
## Decision Brief

**Intent:** Add webhook notifications.
**Outcome:** Webhooks are delivered.
**Scope:** Notifications.

**Implementation Overview:**
- First create data structures.
- Then build infrastructure.
- Then connect business logic.
- Finally add tests.
```

The Decision Brief already leaks task decomposition. A human approval surface should describe the outcome, visible behavior, and scope; layer order belongs only in the Execution Plan.

```markdown
## Task 1: Create all data structures
- [ ] Define `WebhookConfig`
- [ ] Define `WebhookPayload`
- [ ] Define `RetryPolicy`

## Task 2: Create all infrastructure
- [ ] Set up HTTP client wrapper
- [ ] Implement retry logic
- [ ] Add config schema

## Task 3: Create all business logic
- [ ] Implement `WebhookNotifier`
- [ ] Wire into `job_runner.py`

## Task 4: Test everything
- [ ] Write all tests
- [ ] Run integration test
```

**Why this fails**

- The approval surface asks the human to read a layer sequence that does not help approve outcome or scope.
- Task 1 delivers no user-visible behavior and resolves no named uncertainty.
- Task 2 builds infrastructure before proving the completion event can supply a useful payload.
- The first end-to-end verification appears at the end, where wrong assumptions are most expensive to fix.
- A vertical plan would first prove one completed job sends one webhook, then add config and retry behavior.
