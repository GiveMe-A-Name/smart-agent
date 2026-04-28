# Artifact Examples

Positive and negative examples for key judgment calls in technical communication.

---

## Opening Paragraph: Decision / Proposal Artifact

**Positive — recommendation in paragraph 1:**

> We recommend migrating the auth service from session tokens to JWTs by Q3. The current session store (Redis) is a single point of failure that caused 47 minutes of downtime in the March incident; JWTs eliminate the dependency. The main tradeoff is that token revocation becomes slower (up to 15 minutes for blacklist propagation vs. instant with session deletion). This document proposes the migration plan and asks for approval to begin in the next sprint.

The reader knows the recommendation, the problem it solves, the tradeoff, and the ask — before the second paragraph.

**Negative — recommendation buried:**

> The auth service was originally built in 2019 using Redis-backed session tokens. At the time, this was the standard approach for the team. Over the past year, several incidents have highlighted limitations with this approach. In March, a Redis failover caused 47 minutes of downtime. The team has evaluated several alternatives and believes JWT-based auth addresses these concerns. We recommend migrating by Q3.

The recommendation does not appear until the fifth sentence. A reader who stops after paragraph 1 has only context — no decision, no ask, no tradeoff.

---

## Failure Signal: Claim Without Evidence

**Negative — claim drives urgency with no anchor:**

> The current database connection pool is too small and will cause problems as traffic grows. We need to increase it soon.

No numbers, no timeline, no evidence. "Too small," "problems," and "soon" cannot be acted on.

**Positive — same claim with evidence:**

> The connection pool is currently sized at 20 connections. At peak load (2-4pm UTC), active connections reach 18-19 (P95 over the past 30 days). At current traffic growth (~8% month-over-month), we will saturate the pool within 6 weeks. We recommend increasing it to 50 connections now to avoid a queue buildup that would add ~200ms to every database request.

Each claim that drives urgency has a number, a condition, or observed evidence.

---

## Boundary: When NOT to invoke this skill

**Scenario: a Slack message to a teammate**

> "hey, the deploy pipeline is failing on staging — looks like the config change from this morning. I'm rolling it back now"

This is informal and ephemeral. It does not need audience analysis, vehicle selection, or longevity handling. Do not apply this skill.

**Scenario: an inline code comment**

```python
# retry up to 3 times with exponential backoff
```

This is implementation-level documentation handled by code documentation judgment, not a durable artifact requiring the full skill. Do not apply this skill.

---

## Everyday Technical Writing: PR Description

**Negative — describes the diff instead of supplying context:**

> Added Redis caching layer. Updated UserController to call CacheService before hitting the database. Added cache invalidation on profile update.

This is a summary of the diff. The reviewer can read the diff. It provides no motivation, no risk signal, and no review focus — the reviewer must reconstruct intent from the code.

**Positive — supplies what the diff cannot:**

> Adds Redis caching (5-min TTL) to GET /users/:id to reduce database load at peak. User profile queries account for 38% of database CPU at peak (Datadog, past 30 days). Without this we cannot meet the Q3 latency SLA. Main risk: profiles remain stale for up to 5 minutes after update — product confirmed this is acceptable (Slack #eng-product, 2024-06-12). Review focus: cache key construction at `user_cache.go:42` — must scope by `tenant_id` to avoid cross-tenant leakage.

The reviewer knows why this change was made, what the risk is and who accepted it, and where to focus — before reading a single line of code.

---

## Code Documentation: API Docs

**Negative — documents the implementation, not the contract:**

```python
def create_payment(amount: float, currency: str) -> dict:
    """
    Calls the Stripe API with the given amount and currency,
    converts currency to the Stripe format (e.g., USD → usd),
    and stores the transaction in the payments table.
    Returns the Stripe response as a dict.
    """
```

This describes how the function works internally. When the implementation changes (different payment processor, different storage), this comment lies. A caller has no information about what inputs are valid or what errors to handle.

**Positive — documents the contract a caller depends on:**

```python
def create_payment(amount: float, currency: str) -> dict:
    """
    Creates a payment intent for the given amount.

    amount: positive number in major units (e.g., 10.00 for $10.00)
    currency: ISO 4217 code (e.g., "USD", "EUR")

    Returns: {"id": str, "status": "pending"|"succeeded"|"failed", "amount": float}
    Raises: PaymentValidationError if amount <= 0 or currency is unrecognized.
    Raises: PaymentGatewayError if the payment processor is unreachable.
    """
```

The implementation can be rewritten entirely without this comment becoming false. A caller knows exactly what to pass, what to expect, and what errors to handle.

---

## Living vs. Point-in-Time Artifact

**Point-in-time (ADR) — correct handling:**

The ADR records the decision as it was made. When the decision changes, a new ADR is written that supersedes this one. Do not append corrections or update history inline — that corrupts the historical record.

**Living artifact (Runbook) — missing update owner:**

> Runbook: Restarting the Payment Service
> Last updated: 2024-01-15

No owner named. When the restart procedure changes (it will), nobody knows they are responsible for updating this. Future operators follow stale commands.

**Living artifact — correct:**

> Runbook: Restarting the Payment Service  
> Owner: Platform On-Call (rotation lead)  
> Source of truth: this file; update before closing any incident that required a procedure change
