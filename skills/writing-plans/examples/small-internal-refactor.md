# Small: Internal Refactor Without Design Review

## Decision Brief

**Intent:** Remove duplicated date-range normalization so report filters behave consistently and future fixes land in one place.

**Outcome:** The two existing report entry points use the same established normalization behavior without changing accepted inputs or results.

**Scope:** Internal report-filter normalization only; public request fields, validation errors, and query semantics remain unchanged.

**Assessment:** Small refactor. `src/reports/http-filters.ts` and `src/reports/scheduled-filters.ts` contain equivalent normalization copied from the established helper pattern in `src/reports/filter-utils.ts`; callers and tests confirm identical contracts. The main constraint is preserving error and default behavior. Done means both entry points share one normalization implementation while accepted values, defaults, validation errors, and query results remain unchanged.

## Execution Plan

1. Move the shared normalization behavior into `src/reports/filter-utils.ts`, preserving the current accepted values, defaults, and validation errors; update both callers and their focused tests in the same working change.
2. Confirm both entry points retain their existing behavior assertions after the shared helper is adopted.

**Final verification:** `pnpm test tests/reports/http-filters.test.ts tests/reports/scheduled-filters.test.ts && pnpm typecheck` → both suites pass and type checking exits `0`.
