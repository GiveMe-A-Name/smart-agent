# Complete TDD Worked Example: Discount Engine

**Scenario**: Implement a discount engine for an e-commerce order. Rules: orders over $100 get 10% off; orders over $200 get 20% off. Orders at or under $100 get no discount.

This example shows: orientation decision, fake-it technique, triangulation, a refactor moment, an edge-case confirmation, and a design pressure signal.

---

## Step 0: Orientation Decision

*Before writing the first test.*

**Design question audit**: what does this task leave open that writing the first test will force a decision on? In this case: none. The discount rules are fully specified (>$100 → 10%, >$200 → 20%), the function signature is clear (`calculateDiscount(total) → number`), and there are no dependency injection decisions or interface seams to discover. Design is fully pre-decided — stated explicitly here.

This means each test will confirm a pre-known behavior rather than discover a new design constraint. Strict cycle-by-cycle is used below (not batch) to show incremental RED verification; batch would be acceptable here since all behaviors are enumerable upfront.

The algorithm is the hard part — the discount rules, thresholds, and tiers. This calls for **inside-out (Chicago school)**: start with the core domain logic, get it working in isolation, then integrate.

Starting point: a function `calculateDiscount(orderTotal: number): number`.

---

## Cycle 1: No discount for small orders

### RED

```typescript
test('applies no discount to orders at or under $100', () => {
  expect(calculateDiscount(50)).toBe(0);
});
```

Run: FAIL. `calculateDiscount is not defined`.

> Verify the failure is "not defined" — that's correct. If it said something else, the test scaffolding has a problem.

### GREEN

```typescript
function calculateDiscount(total: number): number {
  return 0;
}
```

Run: PASS.

This looks too simple. That's exactly right. We hardcoded `0` — the *fake it* technique. We aren't guessing at the full algorithm. The next test will make `return 0` insufficient, forcing us to write real logic.

### REFACTOR

Nothing to clean up. Move on.

---

## Cycle 2: 10% discount for orders over $100

### RED

```typescript
test('applies 10% discount to orders over $100', () => {
  expect(calculateDiscount(150)).toBe(15);
});
```

Run: FAIL. Expected `15`, received `0`. Correct — `return 0` is now proven wrong for this case.

### GREEN

```typescript
function calculateDiscount(total: number): number {
  if (total > 100) return total * 0.1;
  return 0;
}
```

Run: PASS. Previous test still passes.

### REFACTOR

Nothing to clean up yet.

---

## Cycle 3: 20% discount for orders over $200

*This is triangulation in action: a third data point forces the general rule.*

### RED

```typescript
test('applies 20% discount to orders over $200', () => {
  expect(calculateDiscount(250)).toBe(50);
});
```

Run: FAIL. Expected `50`, received `25` (the `> 100` branch fired). Correct.

### GREEN

```typescript
function calculateDiscount(total: number): number {
  if (total > 200) return total * 0.2;
  if (total > 100) return total * 0.1;
  return 0;
}
```

Run: PASS. All 3 tests pass.

### REFACTOR

The thresholds and rates are magic numbers buried in conditions. The pattern of "tiers with thresholds" is now visible. Extract it:

```typescript
const DISCOUNT_TIERS = [
  { threshold: 200, rate: 0.2 },
  { threshold: 100, rate: 0.1 },
];

function calculateDiscount(total: number): number {
  const tier = DISCOUNT_TIERS.find(t => total > t.threshold);
  return tier ? total * tier.rate : 0;
}
```

Run: All 3 tests pass. The structure changed; the behavior did not. This is what safe refactoring looks like — tests stay green at every step.

> Notice: the refactored form also makes adding a new tier trivial. But YAGNI — we add it only when a test requires it, not because we can.

---

## Cycle 4: Edge case — exactly $100 (boundary confirmation)

### RED

```typescript
test('applies no discount to orders of exactly $100', () => {
  expect(calculateDiscount(100)).toBe(0);
});
```

Run: **PASS immediately.**

Stop. This is not a characterization test (those apply to legacy code before changes). This is a boundary confirmation: the implementation written in Cycle 1 (`total > 100`) incidentally covers exactly $100. The test passed because the behavior was already present, not because the test is wrong.

Verify intent before keeping: does the rule intend "over $100" to exclude exactly $100? Yes — the spec says "over $100". `total > 100` returns `0` for `total === 100`. The test is correct.

Verdict: keep the test as documentation of the boundary condition. Add a comment:

```typescript
// Documents the boundary: exactly $100 is not "over $100"
test('applies no discount to orders of exactly $100', () => {
  expect(calculateDiscount(100)).toBe(0);
});
```

---

## Design Pressure: A New Requirement

The next requirement arrives: *"Loyalty program members get an additional 5% off on top of the base discount."*

You start writing the test:

```typescript
test('loyalty members get additional 5% on top of base discount', () => {
  const discount = calculateDiscount(150, ???);
});
```

**The test immediately surfaces a design question**: how does the function receive membership status?

**Option A**: `calculateDiscount(150, true)` — a boolean parameter. Callers will write `calculateDiscount(amount, user.isLoyaltyMember)` and the intent is opaque. Boolean arguments that control behavior are a design smell.

**Option B**: `calculateDiscount(150, { loyaltyMember: true })` — a config object. Better readability, but now `calculateDiscount` has two responsibilities: computing the discount tier and applying loyalty bonuses.

**Option C**: Separate concerns. `calculateDiscount` stays pure — it only applies tier rules. A new function `applyLoyaltyBonus(baseDiscount, member)` handles the loyalty logic.

```typescript
test('loyalty bonus adds 5% to base discount', () => {
  expect(applyLoyaltyBonus(15, { loyaltyMember: true })).toBe(15 * 1.05);
});
```

This test is trivial to write and trivial to implement. The friction from Option A and B was TDD telling you the design was wrong.

**Lesson**: TDD did not slow down adding the feature. It forced the decision about *where* the responsibility lives before any code was written. Without writing the test first, you might have added a boolean parameter, shipped it, and refactored it three months later.

---

## Summary

| Cycle | Technique used | What it taught |
|-------|---------------|----------------|
| 1 | Fake it (hardcode `0`) | Don't pre-build the algorithm; let tests force it |
| 2 | Simple conditional | Second test made the hardcode insufficient |
| 3 | Triangulation | Third test forced the general tier pattern to emerge |
| Refactor | Safe restructure under green | Changed structure without changing behavior |
| 4 | Boundary confirmation | Passing immediately = verify intent, then keep as docs |
| New requirement | Design pressure | Hard to write test = API smell → separate concerns |

The complete feature took 4 short cycles. Each was independently verifiable. The refactor was safe because every step had green tests. The design pressure from the fifth requirement produced a cleaner architecture with less code than the alternatives.
