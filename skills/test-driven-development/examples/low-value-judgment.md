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

## Case 7: Small Change — Size Is Not the Criterion

Two one-line changes. One skips TDD; the other does not. The difference is not size.

**Change A — Correct Skip:**
```python
# Before
raise ValueError("Invlaid input: value must be positive")
# After
raise ValueError("Invalid input: value must be positive")
```
No test asserts on this message string. The caller-observable behavior (exception type, when it fires) is unchanged. No new execution path.

**Correct:** Skip TDD. State: "Typo fix in error message — no caller-observable behavior change."

---

**Change B — TDD Required:**
```python
# Before
if age > 18:
    allow_access()
# After
if age >= 18:
    allow_access()
```
One character, but a caller passing `age=18` now observes a different outcome. The behavior contract changed.

**Wrong:** "It's just one character, I'll skip TDD."
**Correct:** Apply TDD. Test needed: `age == 18` now grants access (boundary case).

---

**The criterion:** Does this change modify what any caller can observe? Yes → TDD. No → skip, and name why.

---

## Case 9: Both TDD Values Absent — Correct Skip

**The rule:** Skip TDD when both core values are simultaneously absent:
- **Design feedback = 0** — no API contract being designed for callers; writing the test first surfaces no design questions
- **Regression protection = 0** — no system depends on this behavior long-term; lifecycle is uncertain or finite

When only one is absent, TDD may still apply. Both must be absent to skip.

**Example:**
```python
# backfill_missing_user_avatars.py
def run():
    users = db.query("SELECT id FROM users WHERE avatar_url IS NULL")
    for user in users:
        avatar_url = generate_default_avatar(user.id)
        db.execute("UPDATE users SET avatar_url = ? WHERE id = ?", avatar_url, user.id)
```
Run once to fix a data gap, then discarded. No other code calls `run()`. No system observes or depends on its behavior.

- Design feedback = 0: there is no API contract being shaped — no caller will ever use this interface
- Regression protection = 0: once run, it will never be executed again; no dependent will break if the logic changes

**Correct:** Skip TDD. State: "No API contract being designed, no system depends on this behavior — design feedback and regression protection both N/A."

---

## Pattern: How to Name a Skip

When skipping TDD, the explanation must identify a no-behavior criterion:

| Acceptable | Not acceptable |
|-----------|---------------|
| "Pure config — JSON file, no logic" | "Simple" |
| "Trivial forwarding — single delegation, no branching" | "I'll test after" |
| "Type declaration only — no runtime behavior" | "Already mentally tested it" |
| "Both TDD values absent — no API contract being designed, no system depends on this behavior long-term" | "This is different because..." |
| "No behavior contract change — typo fix in log message, no test asserts on string, existing tests cover the path" | "It's a small change" |
