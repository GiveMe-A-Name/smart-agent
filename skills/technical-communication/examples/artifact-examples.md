# Artifact Examples

Read the matching case when the artifact shows that pattern. Apply the judgment before polishing prose.

---

## Proposal Opening - Put the Decision Before Context

Read this when drafting or revising an RFC, design doc, proposal, review response, or change summary whose opening starts with history, background, or implementation detail.

Bad move:

> The auth service was originally built in 2019 using Redis-backed session tokens. Over the past year, several incidents have highlighted limitations with this approach. In March, a Redis failover caused 47 minutes of downtime. The team evaluated several alternatives and believes JWT-based auth addresses these concerns. We recommend migrating by Q3.

Better move:

> We recommend migrating the auth service from Redis-backed session tokens to JWTs by Q3. Redis session storage caused 47 minutes of downtime during the March failover incident; JWTs remove that runtime dependency. The main tradeoff is slower token revocation, up to 15 minutes through blacklist propagation instead of instant session deletion. This document asks for approval to start the migration next sprint.

Judgment to apply:

- If the artifact asks for approval, review, or alignment, put the recommendation, problem, tradeoff, and ask in paragraph 1.
- Move historical context after the opening unless the context is needed to understand the recommendation itself.
- Do not make the reader infer the ask from background.

---

## Evidence Anchor - Replace Vague Action Claims

Read this when a claim drives urgency, priority, risk, or correctness and uses vague words such as "soon", "too small", "often", "may", "various", or "problematic."

Bad move:

> The current database connection pool is too small and will cause problems as traffic grows. We need to increase it soon.

Better move:

> The connection pool is capped at 20 connections. During peak traffic from 2-4pm UTC, active connections reach 18-19 at P95 over the past 30 days. With current traffic growth of about 8% month over month, the pool will saturate within 6 weeks. Increase the pool to 50 connections to avoid queueing that would add about 200ms to database-backed requests.

Judgment to apply:

- Replace vague urgency with an observed number, condition, timeframe, example, link, or explicit assumption.
- If no anchor is available, qualify the claim as an assumption or remove it from the recommendation.
- Do not let a recommendation depend on adjectives the reader cannot verify.

---

## PR Description - Supply What the Diff Cannot

Read this when writing or revising a PR description that describes changed files or implementation steps but does not explain why the change exists, what risk matters, or where review should focus.

Bad move:

> Added Redis caching layer. Updated UserController to call CacheService before hitting the database. Added cache invalidation on profile update.

Better move:

> Adds Redis caching with a 5-minute TTL to `GET /users/:id` to reduce peak database load. User profile reads account for 38% of database CPU during peak traffic over the past 30 days. Main risk: profiles can remain stale for up to 5 minutes after update; product accepted this for non-critical profile fields. Review focus: cache key construction in `user_cache.go` must include `tenant_id` to prevent cross-tenant data exposure.

Judgment to apply:

- Do not repeat file-level changes that are already visible in the diff.
- Provide motivation, evidence, user or system impact, accepted risk, test or rollout evidence, and review focus.
- Name the riskiest file, behavior, or contract so review effort goes to the right place.

---

## API Docs - Document the Caller Contract

Read this when documenting a function, endpoint, SDK method, CLI command, or public interface.

Bad move:

```python
def create_payment(amount: float, currency: str) -> dict:
    """
    Calls Stripe with the given amount and currency, converts currency to the Stripe format, stores the transaction in the payments table, and returns the Stripe response.
    """
```

Risk signal: the text names internal providers, storage, or implementation sequence while omitting valid inputs, outputs, and errors. Move those internals out unless callers must act on them.

Better move:

```python
def create_payment(amount: float, currency: str) -> dict:
    """
    Creates a payment intent.

    amount: positive number in major units, such as 10.00 for $10.00.
    currency: ISO 4217 code, such as "USD" or "EUR".

    Returns: {"id": str, "status": "pending"|"succeeded"|"failed", "amount": float}
    Raises: PaymentValidationError if amount <= 0 or currency is unrecognized.
    Raises: PaymentGatewayError if the payment provider is unreachable.
    """
```

Judgment to apply:

- Document what callers can rely on: valid inputs, outputs, errors, side effects, compatibility, and examples.
- Do not document internal providers, storage, conversion steps, or implementation sequence unless callers must act on that detail.
- If the implementation can change without changing caller behavior, keep it out of the contract docs.

---

## Artifact Lifetime - Supersede Snapshots, Maintain Living Docs

Read this when editing an ADR, postmortem, runbook, API reference, README, or architecture overview.

Bad move for an ADR:

> Update ADR-004 to say we now use JWTs instead of Redis sessions.

Risk signal: the edit rewrites a point-in-time decision as if it were a living artifact. Supersede it instead so future readers keep the original decision context.

Better move for an ADR:

> Create a new ADR: "ADR-007: Migrate auth sessions from Redis to JWTs." Include the new decision, current constraints, alternatives considered, consequences, and `Supersedes: ADR-004`. If repository convention allows, add only a short pointer in ADR-004: `Superseded by ADR-007`.

Bad move for a runbook:

> Restart the payment service with `kubectl rollout restart deploy/payment`.

Risk signal: the runbook gives a command but no owner, source of truth, review trigger, expected result, or escalation path. Add the maintenance path before treating it as publishable.

Better move for a runbook:

> Owner: platform on-call rotation. Source of truth: `deploy/k8s/payment-service`. Review trigger: any change to deployment name, namespace, readiness check, or rollback procedure. First check: `kubectl -n payments get deploy/payment -o wide`. Restart command: `kubectl -n payments rollout restart deploy/payment`. Expected result: rollout completes within 5 minutes; escalate to platform incident channel if it does not.

Judgment to apply:

- Treat ADRs and postmortems as snapshots. Supersede them instead of rewriting their historical decision or incident record.
- Treat runbooks, READMEs, API docs, and architecture overviews as living artifacts. Include an owner, source of truth, review trigger, repository convention, or explicit assumption before materially revising them.
- If the user asks for an edit that conflicts with artifact lifetime, surface the conflict and choose the shape that preserves future reader trust.
