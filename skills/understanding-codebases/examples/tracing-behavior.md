# Tracing Behavior

This example shows delegation chain tracing: following a lookup past the entry point to the code that actually makes decisions.

---

## Task

"Where does the auth token get validated?"

**Investigation strategy**: entry point identification + delegation chain tracing.

---

## Investigation

**Hypothesis**: Auth validation likely starts in request middleware. Search for auth-related middleware.

**Lookup 1**: Read `middleware/auth.ts`. It has a `validateToken` call that delegates to `services/token.ts`. This grounds the entry path, but the file delegates — not yet the behavior-changing code.

**Lookup 2**: Read `services/token.ts` — it performs signature checks and expiry validation. This is the behavior-changing code.

---

## Result

`middleware/auth.ts` is the entry point and orchestration layer; `services/token.ts` contains the actual validation logic. Any deeper delegation not directly confirmed stays labeled as hypothesis.

---

## What This Prevents

Stopping at the first plausible file. Treating a function name like `validateToken` as proof of where validation really happens — rather than following the delegation to confirm.
