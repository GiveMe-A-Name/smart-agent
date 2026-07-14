# Bug-Fix Working Model

## Task

“Fix checkout so expired coupons are rejected.”

## Investigation

1. Trace the reported checkout path: `routes/checkout.ts` → `CheckoutService.applyCoupon` → `CouponPolicy.canApply`.
2. `CouponPolicy.canApply` currently decides eligibility from `disabled` and `minimum_total`; it does not read expiry.
3. The schema and model show nullable UTC `expires_at`; fixtures and tests establish that `null` means no expiry.
4. Time comes from `CheckoutClock.now`, which tests can freeze.
5. Checkout, API validation, and admin preview consume the same policy result.

## Working model

- **Symptom boundary:** checkout accepts a coupon after its expiry; creation and admin editing are outside the report.
- **Current design:** routes orchestrate checkout, while `CouponPolicy` owns shared eligibility decisions.
- **Runtime cause:** the checkout path reaches the policy, but the policy has no expiry decision.
- **Data contract:** `expires_at` is nullable UTC; `null` currently means no expiry.
- **Consumers:** changing policy behavior is observable in checkout, API validation, and admin preview.
- **Reliable change point:** `CouponPolicy.canApply` is the existing eligibility decision point; an admin form is not on the checkout path.
- **Verification surface:** frozen-clock tests for expired and non-expiring coupons, plus affected consumer coverage.
- **Intent:** shared eligibility ownership is documented, inferred, or unknown depending on ADRs, tests, and history.

The investigation establishes where behavior and risk live. It does not yet choose the comparison semantics, compatibility treatment, or whether the policy structure should change.

## Failure pattern

Adding a warning to an admin form after searching only for `expires_at` changes display behavior, not checkout acceptance, and ignores the nullable contract.
