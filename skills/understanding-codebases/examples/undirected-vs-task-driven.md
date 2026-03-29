# Undirected Browsing vs. Task-Driven Investigation

This contrast pair shows the difference between exploring a codebase
without purpose and investigating with a specific task hypothesis.

---

## Bad Case: Undirected Browsing

**Task:** "Add a retry mechanism to the email sending service"

**Step 1:** Read the project README. It describes the app in general terms.
**Step 2:** Read the directory listing of `src/`. Note 12 subdirectories.
**Step 3:** Read `src/config/index.ts` to understand configuration.
**Step 4:** Read `src/database/connection.ts` to understand database setup.
**Step 5:** Read `src/services/email.ts` — finally found the email service.
**Step 6:** Still unclear how to add retry. Read `src/services/user.ts` to
see how other services work.

**What went wrong:**

Steps 1–4 were curiosity-driven. None of them tested a hypothesis about
where email sending happens or how the codebase handles retries. The
agent gathered general context that does not help answer the specific
question. By step 5 it found the right file, but wasted budget on
unrelated code — and then step 6 explored yet another service without
first checking whether a retry pattern already exists in the codebase.

---

## Corrected Case: Task-Driven Investigation

**Task:** "Add a retry mechanism to the email sending service"

**Hypothesis 1 — Where is the email sending code?**
Search for "email" in filenames and imports. Find `src/services/email.ts`.
Read it: it calls `transporter.sendMail()` with no retry logic. This
grounds where the change goes.

**Hypothesis 2 — Does the codebase already have a retry pattern?**
Search for "retry" across the codebase. Find `src/utils/retry.ts` — a
generic retry wrapper with configurable backoff. Also find it used in
`src/services/payment.ts` wrapping the payment gateway call. This
grounds how retries are done here: use the shared utility, matching
the pattern in the payment service.

**Result:** The email service at `src/services/email.ts` needs retry
logic. The codebase provides `src/utils/retry.ts` as the standard
approach, demonstrated by `src/services/payment.ts`. The change should
wrap `transporter.sendMail()` with the shared retry utility, following
the payment service pattern.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Starting move | Read general project docs | Search for email-related code directly |
| Second move | Browse unrelated config/database files | Search for existing retry patterns |
| Pattern discovery | Skipped — never checked for existing retry utilities | Found shared retry utility and usage example |
| Result after similar effort | Found the right file but no implementation strategy | Found the file, the utility, and a reference pattern |

Task-driven investigation asks "what do I need to know to do this task?"
and lets each hypothesis lead to the next. Undirected browsing asks
"what is in this repo?" and hopes to stumble onto something useful.
