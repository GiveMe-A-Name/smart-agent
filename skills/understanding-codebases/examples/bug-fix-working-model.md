# Bug Fix Working Model

This example shows a bug fix where the tempting edit is a local conditional, but the correct model also needs data shape, time/config source, consumers, and verification.

---

## Task

"Fix checkout so expired coupons are rejected."

**Investigation strategy**: symptom boundary + entry path + data/state tracing + consumer and verification model.

---

## Investigation

**Step 1 — Bound the symptom**: A failing test or report says checkout accepts a coupon after `expires_at`. The behavior to change is checkout acceptance, not coupon creation or admin editing.

**Step 2 — Trace the entry path**: `routes/checkout.ts` calls `CheckoutService.applyCoupon`, which delegates to `CouponPolicy.canApply`.

**Step 3 — Find the behavior owner**: `CouponPolicy.canApply` checks `disabled` and `minimum_total` but does not check expiry. This is the first behavior-changing point on the checkout path.

**Step 4 — Trace data and state**: `Coupon.expires_at` is nullable in the database migration, parsed as UTC in `models/coupon.ts`, and existing fixtures include both `null` and timestamp values. `null` means no expiry. Checkout receives current time from `CheckoutClock.now`, and tests can freeze that clock.

**Step 5 — Check consumers**: Admin preview also calls `CouponPolicy.canApply`, but only for displaying eligibility. API coupon validation and checkout both depend on the same policy, so a policy-level fix updates both paths.

**Step 6 — Identify verification**: Existing checkout tests cover disabled coupons and minimum totals. Add an expired-coupon checkout test and a non-expiring coupon test using the existing coupon fixture builder.

---

## Working Model

- **Task boundary**: reject expired coupons at checkout; do not change coupon creation or admin editing.
- **Entry path**: checkout route to `CheckoutService.applyCoupon` to `CouponPolicy.canApply`.
- **Behavior owner**: `CouponPolicy.canApply`.
- **Data model**: `Coupon.expires_at` is nullable, parsed as UTC, and `null` means no expiry.
- **Time/config source**: expiry comparison uses `CheckoutClock.now`, which tests can freeze; no feature flag override is present on the checkout path.
- **Consumer model**: checkout, API coupon validation, and admin preview use the same policy.
- **Verification path**: checkout tests for expired and non-expiring coupons using the existing fixture builder.

The supported change is in `CouponPolicy.canApply`: reject coupons whose non-null `expires_at` is before the current checkout time, preserving `null` as no expiry.

---

## Bad Case: Editing the First Local Conditional

**Step 1:** Search for `expires_at` and find a check in `admin/coupons/form.ts`.

**Step 2:** Add a UI warning for expired coupons.

**What went wrong:**

The agent found a field name but did not trace the checkout path. The admin form is not the behavior owner for checkout acceptance. The fix changes display behavior while checkout still accepts expired coupons.

It also missed that `expires_at` is nullable and that `null` means no expiry. A naive comparison could reject coupons that should never expire.

| | Bad case | Corrected case |
|---|---|---|
| Entry path | Field search only | Checkout route to service to policy |
| Behavior owner | Admin form | `CouponPolicy.canApply` |
| Data model | Timestamp assumed required | Nullable UTC timestamp; `null` means no expiry |
| Consumer model | Checkout not traced | Checkout, API validation, admin preview considered |
| Verification | UI warning not tested against checkout | Checkout tests for expired and non-expiring coupons |
