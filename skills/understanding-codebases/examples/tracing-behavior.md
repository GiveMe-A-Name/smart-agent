# Tracing Behavior

This example shows delegation chain tracing: following a lookup past the entry point to the code that actually makes decisions before choosing where to edit.

---

## Task

"Tighten auth token expiry validation."

**Investigation strategy**: entry point identification + delegation chain tracing.

---

## Investigation

**Hypothesis**: Auth validation likely starts in request middleware. Search for auth-related middleware to find the execution path.

**Lookup 1**: Read `middleware/auth.ts`. It has a `validateToken` call that delegates to `services/token.ts`. This grounds the entry path, but the file delegates — not yet the behavior-changing code.

**Lookup 2**: Read `services/token.ts` — it performs signature checks and expiry validation. This is the behavior-changing code.

**Lookup 3**: Read the token type and tests. The token payload contains an `exp` timestamp, validation reads the current time through `Clock.now`, and token tests already freeze the clock for expiry cases.

**Lookup 4**: Search callers of `validateToken`. Request middleware is the only production caller; token tests call the service directly.

---

## Working Model

- **Entry path**: request middleware in `middleware/auth.ts` invokes token validation.
- **Behavior owner**: `services/token.ts` performs signature checks and expiry validation.
- **Data and time model**: token expiry comes from the payload `exp` timestamp; current time comes from `Clock.now`, which tests can freeze.
- **Consumer model**: request middleware is the production caller; service tests cover direct validation behavior.
- **Change point**: stricter expiry behavior belongs in `services/token.ts`, not in the middleware wrapper.
- **Verification path**: add or update a service-level expiry test and keep middleware behavior covered through the existing auth middleware test.
- **Unknowns**: any deeper delegation not directly confirmed stays labeled as outside the current change.

---

## What This Prevents

Stopping at the first plausible file. Treating a function name like `validateToken` as proof of where validation really happens — rather than following the delegation to confirm.

---

## Bad Case: Stopping at the Facade

**Task:** "Tighten auth token expiry validation."

**Lookup 1:** Read `middleware/auth.ts`. It has a `validateToken` call that delegates to `services/token.ts`.

**Conclusion:** "Add the expiry change in `middleware/auth.ts` because it calls `validateToken`."

**What went wrong:**

The agent stopped at the entry point and treated the function name as proof of behavior. `middleware/auth.ts` is an orchestration layer — it calls `validateToken` but does not contain the validation logic. The actual signature checks and expiry validation live in `services/token.ts`, which was not read.

If a change to validation behavior is needed — stricter expiry, different signature algorithm — making it in `middleware/auth.ts` is in the wrong place. The change would need to be in `services/token.ts`.

**The distinction:**

A function name (`validateToken`) and a call site (`middleware/auth.ts`) confirm that validation is invoked here — not that the logic lives here. Stopping at the first plausible file puts the working model one delegation layer away from the real change point.

| | Bad case | Corrected case |
|---|---|---|
| Lookups performed | 1 (entry point only) | 2 (entry point + delegate) |
| Change point | `middleware/auth.ts` (wrong layer) | `services/token.ts` (behavior-changing code) |
| Working model quality | Function name treated as proof | Actual signature check and expiry logic read |
| Risk | Change in wrong file; behavior unchanged | Change in correct location |
