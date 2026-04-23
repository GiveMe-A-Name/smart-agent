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
