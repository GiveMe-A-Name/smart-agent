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

---

## Bad Case: Stopping at the Facade

**Task:** "Where does the auth token get validated?"

**Lookup 1:** Read `middleware/auth.ts`. It has a `validateToken` call that delegates to `services/token.ts`.

**Conclusion:** "Auth validation happens in `middleware/auth.ts` via the `validateToken` function."

**What went wrong:**

The agent stopped at the entry point and treated the function name as proof of behavior. `middleware/auth.ts` is an orchestration layer — it calls `validateToken` but does not contain the validation logic. The actual signature checks and expiry validation live in `services/token.ts`, which was not read.

If a change to validation behavior is needed — stricter expiry, different signature algorithm — making it in `middleware/auth.ts` is in the wrong place. The change would need to be in `services/token.ts`.

**The distinction:**

A function name (`validateToken`) and a call site (`middleware/auth.ts`) confirm that validation is invoked here — not that the logic lives here. Stopping at the first plausible file produces a candidate change point that is off by one delegation layer.

| | Bad case | Corrected case |
|---|---|---|
| Lookups performed | 1 (entry point only) | 2 (entry point + delegate) |
| Candidate change point | `middleware/auth.ts` (wrong layer) | `services/token.ts` (behavior-changing code) |
| Evidence quality | Function name as proof | Actual signature check and expiry logic read |
| Risk | Change in wrong file; behavior unchanged | Change in correct location |
