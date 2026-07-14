# Tracing Behavior

## Task

“Tighten auth token expiry validation.”

## Investigation

1. `middleware/auth.ts` is the request entry point and delegates to `validateToken`.
2. `services/token.ts` performs signature and expiry decisions; the middleware only translates the result into a response.
3. The payload type defines `exp`; validation reads time through `Clock.now`; service tests freeze that clock.
4. Reference search finds the middleware as the only production caller and tests as direct consumers.
5. History shows token checks were centralized to keep transport handling separate from token policy. This makes the responsibility split documented if the commit or design note says so; otherwise it is only inferred from structure.

## Working model

- **Current design:** middleware owns request adaptation; the token service owns token validity decisions.
- **Runtime path:** request → auth middleware → token service → signature and expiry checks → translated response.
- **Reliable change point:** the existing expiry decision is in `services/token.ts`; middleware is a caller, not the behavior owner.
- **Contracts and consumers:** the service returns the validation result consumed by middleware; tests exercise expiry through a controllable clock.
- **Design intent:** central policy versus transport adaptation, labeled according to the available historical evidence.
- **Verification surface:** service expiry tests plus the existing middleware translation test.
- **Unknowns:** any dynamically registered callers not covered by the reference search.

This model locates responsibility and constraints. It does not decide the new expiry rule or whether the service needs refactoring.

## Failure pattern

Stopping at `middleware/auth.ts` confuses an entry point with a behavior owner. A function call proves that validation is invoked there, not that the caller implements it.
