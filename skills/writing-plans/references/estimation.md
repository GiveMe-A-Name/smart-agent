# Estimation Reference

Read this reference only when the user asks for effort, duration, staffing, or scheduling estimates, or when repository convention requires estimates in plans.

## Communicate Uncertainty

Use a range plus the evidence that controls it, not a point estimate presented as fact:

```text
2–5 days; likely 3 if the payment API supports idempotency keys, otherwise the
deduplication path adds roughly 2 days.
```

Name the fast-path and slow-path conditions. Widen the range when the domain is unfamiliar, tests are weak, external systems are uncontrolled, or no repository precedent exists.

Prefer measurement when it is cheap: benchmark a migration on representative data or run a bounded spike instead of estimating behavior that can be observed directly.

When an early task invalidates the assumption that drove the range, update remaining estimates and record the changed premise in the Execution Log.
