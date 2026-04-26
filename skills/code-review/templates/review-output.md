# Code Review Output Template

Use this as the default structure when it helps. You may adapt section names or ordering if the review still states the Layer 0-1 assessment, concrete issues, and a verdict. Omit a section only if there are genuinely no findings at that level — do not omit to appear positive.

---

## Change Assessment

[Summarize what this change does in 1-2 sentences, in your own words. Write from the PR description, commit messages, and file list — before deep implementation reading. Layer 0 and Layer 1 assessments here are gates: if they reveal a blocking problem, stop and surface it before recording implementation findings.]

**Change justification (Layer 0):** [Does this change solve a real problem? Is the direction correct? Is the scope appropriate? If there are concerns at this level, state them here — they are likely the most important findings in the review. Written before opening implementation files.]

**Design and approach (Layer 1):** [Is this the right solution? Are there simpler alternatives? Is it over/under-engineered? What are the second-order effects? Does it improve or degrade overall system health? Written after reading the main design file, before cataloging Layer 2-3 findings.]

## Strengths

[Optional. Include when there is something concrete worth preserving or repeating.]

[What is well done — be specific. Cite file:line or name the pattern.
Examples: "Clean error propagation in auth.ts:42-58", "Tests cover the failure path at handler.test.ts:91", "Chose the simplest approach that solves the problem"]

## Issues

### Critical (Must Fix Before Merge)

[Layer 0-1 problems that invalidate the change, security vulnerabilities, data loss risks, broken existing functionality.
If none: omit this section.]

**1. [Short title]**
- **Layer:** [0: Change Justification / 1: Design / 2: Implementation / 3: Maintainability]
- **File:** `path/to/file.ts:line`
- **Lens/Standard:** [Optional. Name the specialized lens or user-defined standard when it is what makes this finding defensible.]
- **What:** [Specific description of the problem]
- **Why:** [What breaks, what risk is introduced, what second-order effect is missed]
- **Fix:** [How to address it — for Layer 0-1 issues, this may mean rethinking the approach]

### Important (Should Fix)

[Design concerns, missing error handling, unmet requirements, test gaps, unaddressed caller impact.
If none: omit this section.]

**1. [Short title]**
- **Layer:** [0 / 1 / 2 / 3]
- **File:** `path/to/file.ts:line`
- **Lens/Standard:** [Optional. Name the specialized lens or user-defined standard when it is what makes this finding defensible.]
- **What:** [Specific description]
- **Why:** [Impact on correctness, maintainability, or reliability]
- **Fix:** [Suggestion, if not obvious]

### Minor (Nice to Have)

[Style, naming, documentation, optimization opportunities.
If none: omit this section.]

**1. [Short title]**
- **File:** `path/to/file.ts:line`
- **Lens/Standard:** [Optional.]
- **What / Why / Fix:** [Can be combined for brevity]

## Automated Checks

[Include when the repository exposes merge-gating or touched-area checks for the diff. Record each check as Passed / Failed / Not run.]

- [Check name]: [Passed / Failed / Not run]

## Verdict

**Verdict:** `Ready to merge` / `Not ready` / `Ready with fixes`

**Reasoning:** [One or two sentences of technical justification. Name which layer and which specific issues drive the verdict. If the verdict is `Not ready`, state what must change before re-review.]

---

## Example

### Change Assessment

This change adds rate limiting to the public search API endpoint to prevent abuse and reduce load on the database.

**Change justification (Layer 0):** Valid — production metrics show search endpoint is 60% of DB load, and there have been two abuse incidents this quarter. Rate limiting is an appropriate response.

**Design and approach (Layer 1):** The per-IP token bucket approach is reasonable for this use case. However, the rate limit configuration is hardcoded rather than configurable, which will require a code change and deploy to adjust thresholds. Given that this is a first implementation and optimal thresholds are unknown, this should be configurable.

### Strengths
- Clean separation of rate limiter from request handler (middleware.ts:15-42)
- Comprehensive test coverage: 12 cases including burst handling and token refill
- Clear error response with Retry-After header (handler.ts:85-92)

### Issues

#### Critical

**1. Race condition in distributed token bucket**
- **Layer:** 2: Implementation
- **File:** `ratelimit.ts:45-52`
- **What:** Read-then-write on the token count is not atomic. Under concurrent requests from the same IP, multiple requests can pass the check before any decrement is written.
- **Why:** Under high concurrency, the rate limiter becomes ineffective — exactly the scenario it exists to prevent.
- **Fix:** Use Redis MULTI/EXEC or a Lua script to make the check-and-decrement atomic.

#### Important

**1. Hardcoded rate limit thresholds**
- **Layer:** 1: Design
- **File:** `ratelimit.ts:8-10`
- **What:** Rate limit (100 req/min) and burst size (20) are constants, not configurable.
- **Why:** Optimal thresholds are unknown until production observation. Changing them currently requires a code change and deploy, which is too slow for responding to an active abuse incident.
- **Fix:** Move to environment variables or a runtime config service.

**2. No rate limit bypass for internal services**
- **Layer:** 2: Implementation
- **File:** `middleware.ts:22`
- **Lens/Standard:** Performance
- **What:** Rate limiting applies to all requests including internal service-to-service calls.
- **Why:** Internal batch jobs that call the search API will be rate limited, breaking nightly indexing.
- **Fix:** Add a bypass mechanism for authenticated internal callers (e.g., check for internal service token).

#### Minor

**1. Magic number for token refill interval**
- **File:** `ratelimit.ts:30`
- **What / Why / Fix:** `60000` should be a named constant `TOKEN_REFILL_INTERVAL_MS` for readability.

### Automated Checks

- Unit tests: Passed
- Lint: Not run

### Verdict

**Verdict:** `Not ready`

**Reasoning:** The race condition in the token bucket (Critical, Layer 2) defeats the purpose of the rate limiter under the exact conditions it's designed to handle. The hardcoded thresholds (Important, Layer 1) will make operational response to abuse incidents unnecessarily slow. Fix both before re-review.
