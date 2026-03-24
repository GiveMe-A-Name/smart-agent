# Fake New Hypothesis

This contrast pair illustrates the most common misapplication of the two-lookup rule:
spending a second lookup on a different file but testing the same hypothesis —
"maybe the logic is in a nearby place I haven't checked yet."

---

## Bad Case

**Task:** "Where does rate limiting get enforced?"

**Lookup 1:** Read `middleware/rate-limit.ts`. It imports `checkRateLimit` from
`utils/rate-limit-check.ts` but that file is not present in the repo.

**Lookup 2:** Read `middleware/rate-limit-v2.ts` — a similar file noticed in the
same directory. It also references `checkRateLimit` from `utils/rate-limit-check.ts`,
which is still absent.

**What went wrong:**

The second lookup changed files but not the hypothesis. Both lookups were testing
"is the enforcement in a middleware file?" — and both confirmed the same absence
(the util is missing). The second lookup added no new information.

The correct move after Lookup 1 was to stop and report: the middleware entry point
exists, but the actual enforcement implementation at `utils/rate-limit-check.ts`
is not present in the repository. Further middleware lookups cannot resolve that gap.

---

## Corrected Case

**Lookup 1:** Read `middleware/rate-limit.ts`. It imports `checkRateLimit` from
`utils/rate-limit-check.ts`. The import path is clear.

**Lookup 2:** Read `utils/rate-limit-check.ts` — this tests a materially different
hypothesis: "does the utility contain the enforcement logic, or does it delegate
further?" It follows the delegation chain rather than rechecking the same absence
in a neighboring file.

**Why this is correct:**

The second lookup changed the hypothesis: from "where is the entry point" to
"does this specific utility contain the behavior-changing code." That is a
materially different question worth a lookup. If `utils/rate-limit-check.ts`
contains token bucket logic, sliding window checks, or calls to a concrete
backend, the enforcement path is grounded and investigation can stop.

---

## The Distinction

A new hypothesis changes what you are looking for, not just where you are looking.

| | Bad case | Corrected case |
|---|---|---|
| Lookup 2 question | "Maybe the logic is in a sibling file?" | "Does the delegated module contain enforcement?" |
| Hypothesis change | No — same absence, different location | Yes — different layer, different question |
| Result | Same absence confirmed twice | Either grounded or a clear new gap named |
