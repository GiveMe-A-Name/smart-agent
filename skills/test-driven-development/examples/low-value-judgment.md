# Examples: Judging Low-Value TDD Cases

These examples show where the "no-behavior" judgment holds, where it doesn't, and what rationalization looks like in practice.

---

## Case 1: Trivial Forwarding — Correct Skip

**Code:**
```typescript
export function getUserById(id: string) {
  return db.users.findById(id);
}
```

**Judgment:** This is a one-liner that forwards directly to a well-tested `findById` with no logic, no branching, no transformation. The "trivial forwarding" criterion applies.

**Correct:** Skip TDD. State: "This is trivial forwarding — no logic beyond delegation to `db.users.findById`."

---

## Case 2: Looks Simple, Has Behavior — TDD Required

**Code:**
```typescript
export function getUserById(id: string) {
  if (!id) throw new Error('id required');
  return db.users.findById(id.trim());
}
```

**Judgment:** This has a guard clause and a `.trim()` transformation. Two execution paths exist. The "trivial forwarding" criterion does NOT apply despite the code looking short.

**Wrong:** "This is basically a forwarding function, I'll skip TDD."
**Correct:** Apply TDD. Tests needed: rejects empty id, trims whitespace before lookup.

---

## Case 3: Configuration — Correct Skip

**Code:**
```json
{
  "timeout": 5000,
  "retries": 3,
  "baseUrl": "https://api.example.com"
}
```

**Judgment:** Pure data, no executable logic. "Pure configuration" criterion applies.

**Correct:** Skip TDD. State: "This is a JSON config file with no logic."

---

## Case 4: Config with Logic — TDD Required

**Code:**
```typescript
export const config = {
  timeout: process.env.TIMEOUT ? parseInt(process.env.TIMEOUT) : 5000,
  retries: Math.max(1, parseInt(process.env.RETRIES ?? '3')),
};
```

**Judgment:** This config has runtime logic — parseInt, Math.max, fallback handling. "Pure configuration" criterion does NOT apply.

**Wrong:** "This is just a config object, no tests needed."
**Correct:** Apply TDD. Tests needed: uses default when env var missing, parses env var when present, enforces minimum retries of 1.

---

## Case 5: Rationalization — Wrong Skip

**Scenario:** Agent is about to implement a form validation function.

**Agent thought:** "Validation is straightforward — I know exactly what it should do. I'll write the implementation first and add tests after to verify it works."

**Why this is wrong:**
- "I know exactly what it should do" — that makes it a perfect candidate for writing the test first (the expected behavior is clear)
- "Tests after to verify it works" — tests-after answer "what does this do?", not "what should this do?" They're biased by what was just implemented
- No no-behavior criterion was named

**Correct:** Apply TDD. The behavior is clear → write a failing test first.

---

## Case 6: Tests Written From the Implementation's Perspective — Wrong TDD

**Scenario:** Agent implements a checkout validator. Before writing any code, it writes three tests:

```typescript
test('rejects empty cart', () => {
  expect(validator.validate([])).toEqual({ valid: false });
});
test('checks each item for stock via InventoryService', () => {
  validator.validate([item1]);
  expect(inventoryService.checkStock).toHaveBeenCalledWith(item1.id);
});
test('calls PaymentService.validate before confirming', () => {
  validator.validate(cart);
  expect(paymentService.validate).toHaveBeenCalled();
});
```

**Judgment:** Test 1 asserts on what a caller observes (return value) — valid. Tests 2 and 3 assert on which internal services were called — the agent already knew the implementation and designed the tests *from* that implementation, not from the caller's perspective.

**Gate (from testing-anti-patterns.md):** "Could this test remain identical if I completely rewrote the implementation while keeping the external behavior the same?" For tests 2 and 3: no. If the stock check is moved to a different service call or the payment validation is restructured, these tests break without any observable behavior changing.

**Wrong:** "I wrote all three tests before writing any implementation — TDD is satisfied."
**Correct:** Apply the caller-perspective gate to each assertion. Tests 2 and 3 need rewriting: assert on `result.valid`, `result.errors`, or other caller-observable outcomes — not on internal call sequences.

**Note:** Batch mode is not the problem here. Tests written from the caller's perspective can be written in batch. The problem is tests written from implementation knowledge, regardless of when they were written.

---

## Pattern: How to Name a Skip

When skipping TDD, the explanation must identify a no-behavior criterion:

| Acceptable | Not acceptable |
|-----------|---------------|
| "Pure config — JSON file, no logic" | "Simple" |
| "Trivial forwarding — single delegation, no branching" | "I'll test after" |
| "Type declaration only — no runtime behavior" | "Already mentally tested it" |
| "Throwaway exploration in tmp/ — not reaching production" | "This is different because..." |
