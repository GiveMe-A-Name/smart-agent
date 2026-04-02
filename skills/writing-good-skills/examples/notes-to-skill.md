# Example: Translating Notes Into a Skill

This pair shows the same raw notes turned into a skill two ways.
The bad version looks complete and well-organized. It is not — it is a procedure manual, not a capability guide.
The good version teaches the same knowledge as reusable judgment.

---

## Input: Raw Notes From the User

> My team keeps writing API error handling in totally different ways. Here's what we've agreed on:
> check the status code first; 4xx means don't retry, 5xx and network errors mean retry;
> use exponential backoff with jitter, cap at 60 seconds; check the Retry-After header if it's there;
> always log the attempt number and whether it eventually succeeded; never swallow the exception.

---

## Bad Version

```markdown
---
name: api-error-handling
description: This skill helps you handle API errors. It walks you through reading the
error response, classifying the error type, deciding on a retry strategy, and adding
logging. Use it when you are working on API integration code.
---

# API Error Handling

## Step 1: Check the Status Code

Read the HTTP status code from the response.

## Step 2: Decide Whether to Retry

- 4xx errors: do not retry
- 5xx errors: retry
- Network errors: retry

## Step 3: Apply Exponential Backoff

Use exponential backoff with jitter. Cap delay at 60 seconds.
Check the Retry-After header if present.

## Step 4: Add Logging

Log the attempt number and whether the request eventually succeeded.

## Step 5: Re-raise the Exception

Never swallow the exception. Always re-raise with context.
```

### What Is Wrong

**The description is a workflow summary, not trigger conditions.**
"It walks you through reading the error response, classifying..." tells the agent what steps it will follow, not when to load the skill. An agent scanning skills cannot tell from this description whether to load it for the current task.

**The five steps are the user's notes renumbered.**
The structure maps one-to-one onto the original bullet points. Nothing was generalized. An agent following this will execute the steps in order regardless of context, because the skill encodes a sequence rather than teaching when and why each decision matters.

**No boundary.** The skill never states what it does not own — internal errors, database errors, requirements clarification. The absence of a boundary lets the skill be misapplied and prevents the agent from knowing when to stop.

**No invariants as invariants.** Step 5 ("never swallow the exception") is a hard rule, not a step. Encoding it as step 5 buries it — it will be treated as optional, reached only if steps 1–4 complete.

**Quick test:** Remove the step numbers. What remains is still the same bullet list the user gave, reformatted. The skill added no judgment.

---

## Good Version

```markdown
---
name: api-error-handling
description: "Invoke when implementing or modifying HTTP API error handling — especially when deciding which status codes to retry, configuring backoff, or when retry behavior is inconsistent across the codebase. Cost of unnecessary invocation: brief review of retry policy. Cost of missing: silent error swallowing, inconsistent retry behavior, or retrying non-recoverable failures."
---

# API Error Handling

Write error handling that retries recoverable failures, fails fast on unrecoverable ones,
and never loses the exception context.

## Capability Boundary

This skill owns: classifying HTTP and network errors, retry judgment, backoff configuration,
and logging retry state.

This skill does not own: deciding what error behavior the caller expects (resolve that before
applying this skill), debugging a failing test implementation, or handling non-HTTP errors
such as database or filesystem failures.

## Invariants

- Never retry 4xx status codes. They indicate a client-side problem that retrying will not fix.
- Never swallow exceptions. Always re-raise with context so the caller can handle or log the failure.
- Always use exponential backoff with jitter for retryable errors. Fixed delays worsen rate limiting.
- Always check the Retry-After header before applying the default backoff formula.

## Error Classification

| Category | Status codes | Action |
|---|---|---|
| Client error | 4xx | Fail immediately, do not retry |
| Server error | 5xx | Retry with backoff |
| Network error | Timeout, connection refused, DNS failure | Retry with backoff |

## Retry Judgment

Backoff formula: `min(60, (2 ** attempt) + random jitter)` seconds.

Start at attempt 0. If Retry-After is present on a 429, use that value instead of the formula.
Cap all delays at 60 seconds regardless of attempt count.

## Observability

Log at each retry: the attempt number, the status code or error type, and the delay applied.
Log at final outcome: whether the request succeeded or exhausted retries.
Use WARNING for retried errors that eventually succeed; ERROR for exhausted retries.

## Failure Signals

Stop and re-evaluate if:
- the caller's expected error behavior is unclear before you start
- the API returns errors in a non-standard format that the status code alone does not classify
- you are adding retry logic to a place that already has it
```

### What Is Better

**The description names observable trigger conditions** with explicit cost asymmetry — not steps. An agent scanning skills can match this to the current task ("I'm deciding whether to retry this request") without reading the body, and the costs make the default obvious.

**The invariants are red lines, stated first.** The "never swallow" rule appears as a constraint, not as step 5. An agent cannot reach the implementation sections without passing the invariants.

**The capability boundary states what the skill does not own.** An agent that has already resolved the expected error behavior can proceed. One that has not knows exactly what is missing.

**The error classification is a judgment table, not a sequence.** An agent that already knows the status code can go directly to the retry section. Nothing forces a fixed order.

**Boundary test:** Remove named-skill references — there are none. The skill still teaches a complete, usable capability.

---

## The Core Distinction

A procedure manual answers: *what do I do next?*
A capability guide answers: *what do I own, what must not happen, and what should I notice?*

Notes from a user are almost always a procedure. Turning them into a skill means extracting the judgment — the invariants, the boundary, the decision signals — and making that judgment reusable, not the sequence.
e.
