# Anti-Pattern: Horizontal Slicing

The webhook feature planned as layers instead of vertical behavior:

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
- Task 1 delivers no user-visible behavior and resolves no named uncertainty.
- Task 2 builds infrastructure before proving the completion event can supply a useful payload.
- The first end-to-end verification appears at the end, where wrong assumptions are most expensive to fix.
- A vertical plan would first prove one completed job sends one webhook, then add config and retry behavior.
