---
name: operability
description: "Invoke before shipping any code that will run in production — new services, new features, infrastructure changes, and significant behavioral changes. Cost of unnecessary invocation: a short production-readiness check. Cost of missing: code that works in development but fails, degrades silently, or is impossible to debug in production. When uncertain whether production considerations apply, invoke."
---

# Operability

Evaluate whether code is ready for production — not whether it works, but whether it can be deployed safely, monitored effectively, debugged under pressure, and degraded gracefully when dependencies fail.

## Trigger Logic

**Invocation default**: Code that reaches production without operability review creates an unmonitored, undebuggable, brittle deployment. The cost of an unnecessary operability check is a short review pass. The cost of missing it is a 3am page with no logs, no metrics, and no runbook. When uncertain whether production considerations apply, invoke.

Use when:
- new code, service, or feature is being prepared for production deployment
- existing code is being modified in ways that affect its production behavior (new dependencies, new failure modes, changed data flows)
- infrastructure or deployment configuration is being changed

Do not use when:
- the work is purely local development tooling with no production impact
- the change is confirmed to be test-only code with no production deployment path

## Capability Boundary

This skill owns:
- judging whether code changes are ready for production operation
- identifying gaps in observability, resilience, deployment safety, and debuggability
- deciding what operability work is necessary vs. gold-plating for a given change

This skill does not own:
- the implementation itself — it evaluates production readiness, not correctness
- capacity planning as a long-term exercise
- incident response procedures (though it helps prevent incidents)
- infrastructure architecture decisions

## Invariants

- Code without observability is not production-ready — if you can't see what it's doing, you can't operate it.
- Every new failure mode must have a detection mechanism — silent failures are the most dangerous.
- Every external dependency must have a timeout and a degradation strategy — dependencies WILL fail.
- Deployment must be reversible — if you can't roll back, you can't deploy safely.
- **Before exiting this skill, you MUST complete the Self-Check section at the end.**

---

## Reading the Situation

Before working through judgment dimensions, identify what you're looking at. This table maps observable code situations to the dimensions that need attention first.

| What you see in the code | Start with | Why |
|---|---|---|
| New HTTP/RPC/DB/queue calls to external systems | Dimension 2 (Resilience) | Every new dependency is a new failure mode. Timeouts and fallbacks first. |
| New service, endpoint, or background job | Dimension 1 (Observability) | If it isn't instrumented from the start, it never will be. |
| Database migration, schema change, or data format change | Dimension 3 (Deployment Safety) | Irreversible data changes are the hardest failures to recover from. |
| Error handling: catch blocks, retry logic, fallback paths | Dimension 2 (Resilience) | Verify the error handling actually helps rather than masking failures. |
| New feature with user-visible behavior | Dimension 3 (Deployment Safety) | Needs a kill switch (feature flag) independent of full rollback. |
| Code that will be owned by a different team or on-call rotation | Dimension 4 (Production Readiness) | If someone else debugs it, everything must be externally visible. |
| Log statements being added or modified | Dimension 1 (Observability) | Structured? Sufficient context? Right level? Not leaking sensitive data? |
| Performance-sensitive path (hot loop, high-QPS endpoint) | Dimension 1 (Observability), Dimension 2 (Resilience) | Metrics for load, backpressure for overload. |

---

## Judgment Dimensions

Work through the relevant dimensions for the code change. Not every dimension applies to every change — but skipping a relevant one is how production incidents happen.

### 1. Observability — "Can this code be understood from outside when it fails?"

**Question**: If this code breaks at 3am, can an on-call engineer figure out what happened using only logs, metrics, and traces — without reading the source code or reproducing the issue?

**Key signals**:
- **Structured logging at decision points and error paths.** A log line like `"Error processing request"` is nearly useless. `{"level":"error","service":"payment","request_id":"abc-123","error":"timeout connecting to stripe API after 30s","latency_ms":30042}` enables diagnosis. Look for: does the log carry enough context (request ID, user ID, relevant business identifiers) to reconstruct what happened?
- **Log levels that mean something.** ERROR for things that need human attention. WARN for degraded-but-functioning. INFO for significant business events. DEBUG for development only. If everything is ERROR, nothing is — alert fatigue kills operability.
- **Context propagation through the call chain.** Request IDs, trace IDs, and business identifiers must flow through every layer and appear in every related log entry. If a request touches 3 services and you can't correlate the logs, you can't debug it.
- **Metrics on request rate, error rate, and latency distribution.** For request-driven code, instrument rate, errors, and duration (histograms, not averages — a P50 of 100ms with a P99 of 10s hides behind a 200ms average). For resource-intensive code, instrument utilization and saturation.
- **Trace instrumentation at service boundaries.** Incoming requests, outgoing calls, database queries, queue operations. If a request traverses multiple services, the trace must follow it. Sample intelligently — always trace errors and slow requests.
- **Sensitive data never in logs.** Passwords, tokens, PII, API keys, credit card numbers. If you see these in log statements, stop.
- **Cardinality safety in metric labels.** A metric label with unbounded values (user ID, request path with embedded IDs) will overwhelm the metrics system. Labels must have bounded cardinality.

### 2. Resilience — "What happens when dependencies fail?"

**Question**: For each external dependency this code touches, what happens when that dependency is slow, returns errors, or is completely unreachable? Is the answer "the system degrades gracefully" or "the system falls over"?

**Key signals**:
- **Every external call has a timeout.** No exceptions. A missing timeout means one slow dependency can freeze the entire system by exhausting thread pools and connection pools. The timeout should reflect expected latency, not generosity — if normal response is 100ms, a 30s timeout just delays the inevitable.
- **Connection timeouts vs. read timeouts are distinct.** Can't reach the server is a different failure from server-accepted-but-not-responding. Both need separate limits and may need different handling.
- **Timeouts are handled, not hidden.** A timeout is a signal, not an annoyance. It should be logged (with context), counted (as a metric), and trigger the fallback path. If a timeout is caught and silently swallowed, the code is masking a failure.
- **Circuit breaker or equivalent for repeated failures.** When a dependency is failing, continuing to send it traffic makes recovery harder. Look for: is there a mechanism to stop calling a failing dependency and periodically test recovery? Is the circuit state (open/closed/half-open) observable?
- **Explicit degradation strategy for non-critical dependencies.** Which features are critical (checkout must work) vs. non-critical (recommendations can be omitted)? If a non-critical dependency fails, does the system serve a degraded-but-useful response, or does it fail entirely? Fallback behavior (cached data, defaults, feature removal) should be designed explicitly, not discovered during an incident.
- **Backpressure under overload.** When the system receives more load than it can handle, does it shed load deliberately (429/503) or accept everything and fail randomly? Queues must be bounded — an unbounded queue is a memory leak. Backpressure should be applied close to the edge.

### 3. Deployment Safety — "Can this be deployed and rolled back safely?"

**Question**: If this deployment causes a problem in production, can you revert to the previous version within minutes? What irreversible changes does this deployment make, and how are they protected?

**Key signals**:
- **Database migrations are backward-compatible.** The old code must still work with the new schema. If the migration drops a column, renames a table, or changes a constraint that existing code depends on, rollback becomes impossible. Expand-then-contract: add the new thing, migrate, then remove the old thing in a later deployment.
- **API changes don't break existing clients.** Additive changes (new fields, new endpoints) are safe. Removing fields, changing response shapes, or altering semantics of existing fields are breaking changes even if tests pass — deployed clients don't update when you push code.
- **Feature flags for new user-visible behavior.** A feature flag lets you disable new behavior without a full redeployment. If the only way to turn off a new feature is to roll back the entire deployment, the blast radius of a problem is unnecessarily large.
- **Progressive rollout strategy exists.** Full-traffic deployment maximizes blast radius. Look for: canary deployment (small percentage first, compare error rates), feature flag ramp (internal users, then percentage, then everyone), or blue-green (switch traffic between environments).
- **Health checks are meaningful.** Liveness (is the process running and not deadlocked — restart if fails) and readiness (can this instance serve traffic — remove from load balancer if fails) must be separate. A readiness check should verify the service can reach its critical dependencies — a service that is "up" but can't reach its database should not receive traffic.
- **Data changes are recoverable.** If this change writes data in a new format, can the old format still be read? If the migration is destructive, is there a backup strategy? Data corruption is the failure mode with the longest recovery time.

### 4. Production Readiness — "Could an on-call engineer debug this at 3am?"

**Question**: Imagine someone who has never seen this code is paged about it at 3am. Can they figure out what's wrong and fix it using only the tools, dashboards, logs, and documentation available to them?

**Key signals**:
- **Alerts exist for new failure modes.** If this code introduces a new way to fail, there must be a corresponding alert. "What signal triggers the alert?" should have a concrete answer — not "we'll notice in the dashboard." Alert on symptoms (error rate, latency breach) rather than causes (CPU) when possible.
- **Alert thresholds are calibrated.** An alert that fires constantly gets ignored. An alert that never fires is untested. The threshold should distinguish between "degraded and needs attention" and "broken and needs immediate response."
- **Error context is sufficient for diagnosis.** When an error occurs, can the on-call engineer determine: what operation failed, what input caused it, what dependency was involved, and what the system state was? An error message of "something went wrong" is an operability failure.
- **Runbook or operational documentation exists for complex paths.** Not every code path needs a runbook. But if diagnosis requires specific knowledge (which database to check, which queue to drain, which config to toggle), that knowledge must be written down somewhere the on-call engineer can find it.
- **Dashboards show the right things.** If this code changes what "healthy" looks like (new traffic patterns, new latency profiles, new resource usage), existing dashboards should be updated or new ones created. A dashboard that doesn't reflect reality is worse than no dashboard — it provides false confidence.
- **The diff between "working" and "broken" is visible.** An on-call engineer needs to quickly answer: "Is this broken right now?" If the answer requires reading code or running manual queries, the observability is insufficient.

---

## Scope Decision

After working through the relevant dimensions, decide what operability work is needed:

- **Already adequate**: the code handles the relevant dimensions. Document what you checked and exit.
- **Small gaps**: missing a timeout, a log statement needs context, a metric label is unbounded. Fix inline — these are quick and high-value.
- **Structural gap**: no degradation strategy for a critical dependency, no rollback path for a destructive migration, no observability for a new service. Flag explicitly — this is work that must happen before shipping, not a nice-to-have.
- **Wrong layer**: operability concerns exist but belong in infrastructure (load balancer config, platform-level circuit breakers) rather than application code. Identify the right owner and flag it — don't implement the wrong solution in the wrong layer.

---

## Failure Signals

Stop and reassess if:
- you are about to ship code with no logging at decision points or error paths
- you are adding a new external dependency without a timeout or fallback strategy
- you cannot describe what alerts would fire if this code fails in production
- you are deploying a database migration that is not backward-compatible with the current code
- the deployment has no rollback plan
- you are about to ship a feature with no way to disable it without a full rollback
- error handling consists of swallowing exceptions or logging and continuing without degradation logic
- you are treating observability as "we'll add it later" — later never comes

## Self-Correction Signals

Stop and revise when:
- you are checking operability boxes without reading the actual code paths
- you are assuming existing infrastructure (monitoring, alerting) covers the new code without verifying
- you are conflating "it has logging" with "it has useful logging" — the presence of log statements is not the same as sufficient observability
- you are treating test coverage as a substitute for production observability
- you are recommending operability improvements that don't match the risk level of the change — a minor config tweak doesn't need a runbook, a new payment integration does

## Self-Check Before Exiting

- [ ] Did I work through the relevant judgment dimensions for this change?
- [ ] Does every external dependency have a timeout and a failure handling strategy?
- [ ] Can an on-call engineer diagnose a failure using only logs, metrics, and traces — without reading source code?
- [ ] Is the deployment reversible (rollback, feature flag, or backward-compatible migration)?
- [ ] Are new failure modes detectable through existing or new alerts?
- [ ] Did I catch any self-correction signals?
- [ ] Am I exiting because the code is genuinely production-ready, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
