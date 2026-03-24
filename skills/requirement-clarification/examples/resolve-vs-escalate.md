# Resolve vs. Escalate

This contrast pair illustrates the skill's core judgment call: which unknowns
can the agent answer from evidence, and which genuinely require the user?

The failure mode is offloading avoidable unknowns to the user — asking questions
the agent could have resolved with a targeted lookup.

---

## Scenario

**Request:** "The search feature is broken, can you fix it?"

No further context is provided.

---

## Bad Case

**Agent response:**

> "Before I start, I need to understand the problem better. Can you tell me:
> 1. What specifically is broken — no results, wrong results, or an error?
> 2. What error message do you see?
> 3. Which fields are being searched?
> 4. Should search be case-sensitive?
> 5. Should it match partial words or exact phrases?
> 6. Which users are affected?"

**What went wrong:**

The agent escalated all unknowns to the user without attempting to classify them.

Questions 1–3 and 6 are potentially answerable from repository evidence: failing
tests describe expected behavior, error logs or stack traces may exist, the
search implementation reveals which fields are used. These are `repository fact`
unknowns, not `user intent` unknowns.

Questions 4–5 are mixed: case sensitivity and match type may already be defined
in tests or existing code, or they may genuinely be open product decisions.

The agent asked six questions before doing any investigation. Several of those
answers are sitting in the codebase.

---

## Corrected Case

**Step 1 — Classify unknowns before asking anything:**

| Unknown | Type | Can self-resolve? |
|---|---|---|
| What does the current search code do? | repository fact | Yes — read the implementation |
| What behavior do tests specify? | repository fact | Yes — read the tests |
| What error or symptom is visible? | repository fact | Possibly — check test output or error traces |
| Should it do partial or exact match? | user intent or repo fact | Check tests first; ask if still open |
| Case sensitivity preference | user intent or repo fact | Check tests first; ask if still open |

**Step 2 — Resolve what can be resolved:**

One lookup finds `search.ts`: it uses `===` for exact string matching.
One lookup finds `search.test.ts`: tests expect partial matches and are currently
failing with "expected 3 results, got 0."

**Step 3 — What remains genuinely open:**

The repository evidence answers the "what is broken" and "what the expected
behavior should be" questions. The only genuinely open user intent question is:

> "The tests show search should return partial matches, which currently fails.
> Before I fix the matching logic: should the search be case-sensitive, or
> case-insensitive? I didn't find a test that specifies this."

**Why this is correct:**

The agent resolved all `repository fact` unknowns with two targeted lookups.
Only one genuine `user intent` decision remained. The user receives a focused
question with context, not a broad list.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Unknowns classified before asking | No | Yes |
| Repository fact unknowns resolved first | No | Yes — two lookups |
| Questions sent to user | 6, mostly answerable from code | 1, genuinely open |
| User needs to describe their own codebase | Yes | No |
| Remaining human decision is clear | No | Yes — case sensitivity only |
