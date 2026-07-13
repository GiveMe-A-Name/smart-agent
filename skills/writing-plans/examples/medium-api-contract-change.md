# Medium: Additive API Contract Change

## Decision Brief

**Intent:** Let API clients see a deployment's manual approval state without making a second approval request.

**Outcome:** Deployment responses include the latest approval state while existing clients continue to process deployment lifecycle status unchanged.

**Scope:** Deployment response serialization and approval-state persistence; deployment lifecycle status and existing request fields remain unchanged.

**Visible Changes:**

| Contract | Before | After |
| --- | --- | --- |
| `POST /deployments` response | `id`, `status` | adds nullable `approval_status` |
| `GET /deployments/:id` response | `id`, `status` | adds nullable `approval_status` |

**Approval Needed:** Add a separate nullable field during rollout; do not overload the existing `status` field with approval state.

**Primary Risk:** Mixed-version workers may create deployments before approval state is written, so readers must tolerate `null` throughout the rollout.

## Design Review

`approval_status` is an additive response field with values `pending`, `approved`, `rejected`, or `null`. Existing `status` semantics remain unchanged, and older stored deployments serialize the new field as `null`.

The alternative is adding approval states to the existing deployment `status` enum. It is rejected because current clients use that field for deployment lifecycle decisions; combining lifecycle and approval would be a breaking semantic change even if the JSON shape remained valid.

Rollout order: deploy nullable readers and serializers, then writers that persist approval state. Rollback remains safe because old code ignores the additive stored/response field.

## Situation Assessment

- **Size:** Medium feature spanning deployment serialization, approval persistence, and contract tests.
- **Nature:** Additive API contract and mixed-version persistence migration.
- **Current state:** `src/deployments/serializers/deployment-response.ts` owns both deployment response shapes; `src/approvals/approval-service.ts` owns approval transitions; `src/approvals/approval-status-store.ts` persists optional approval state. Existing nullable-field migrations in the deployment serializer provide the verified pattern.
- **Done state:** Both deployment endpoints expose the nullable approval state, older deployments return `null`, and deployment lifecycle status remains unchanged for existing clients.

## Execution Plan

### Decomposition Strategy

- **Contract first:** prove the additive response remains compatible before changing writers.
- **Mixed-version second:** persist approval state only after readers tolerate `null`.
- **Vertical slices:** each task leaves old deployments and clients valid.

### Exclusions

- No change to deployment lifecycle states.
- No approval-history API in this iteration.
- No backfill; old deployments return `null`.

### Execution Risks And Plan-Internal Responses

- **Risk:** Task 2's writer handoff may differ from the current file map after Task 1 proves the nullable serializer contract, making its file-level instructions stale. **Revision trigger:** refine that handoff after Task 1, provided the separate-field semantics, reader-before-writer rollout, and no-backfill scope remain unchanged.

### Stop Signals

- If compatibility fixtures or a verified client reject the additive response field, stop before Task 1 changes the contract.
- If the status store cannot represent absence as `null` without a backfill or different persistence shape, stop before Task 2 and request a rollout decision.

### File Map

- `src/deployments/serializers/deployment-response.ts`: add the nullable response field without changing deployment lifecycle status.
- `src/approvals/approval-service.ts`: emit approval-state transitions.
- `src/approvals/approval-status-store.ts`: persist optional approval state.
- `tests/deployments/deployment-response.test.ts`: prove additive compatibility for new and old deployments.
- `tests/approvals/approval-status.test.ts`: prove persisted transitions appear in responses.

## Task 1: Prove additive response compatibility

Establish the public contract before persisting new state.

Files: `src/deployments/serializers/deployment-response.ts`; `tests/deployments/deployment-response.test.ts`.

- [ ] Add nullable `approval_status` to both deployment response serializers without changing `status`.
- [ ] Update compatibility fixtures to assert old deployments return `null`.
- [ ] Run `pnpm test tests/deployments/deployment-response.test.ts` → PASS.
- [ ] Commit point: both endpoints expose the additive nullable field.

## Task 2: Persist and expose approval state

Connect the approved contract to approval handling while preserving mixed-version behavior.

Depends on: Task 1's proven nullable response contract.

Files: `src/approvals/approval-service.ts`; `src/approvals/approval-status-store.ts`; `src/deployments/serializers/deployment-response.ts`; `tests/approvals/approval-status.test.ts`; `tests/deployments/deployment-response.test.ts`.

- [ ] Persist `pending`, `approved`, or `rejected` when approval state changes.
- [ ] Map absent state to `null` in deployment responses.
- [ ] Test new and pre-existing deployments across both endpoints.
- [ ] Run `pnpm test tests/deployments/deployment-response.test.ts tests/approvals/approval-status.test.ts` → PASS.
- [ ] Commit point: response state follows persisted approval state without changing lifecycle status.

**Final verification:** `pnpm test && pnpm typecheck` → exits `0`.

## Execution Log

*(append entries here as each task completes - format: `[YYYY-MM-DD] Task N: what was done and why; key decisions; any failures and how they were fixed`)*
