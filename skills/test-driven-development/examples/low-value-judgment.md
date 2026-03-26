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

## Pattern: How to Name a Skip

When skipping TDD, the explanation must identify a no-behavior criterion:

| Acceptable | Not acceptable |
|-----------|---------------|
| "Pure config — JSON file, no logic" | "Simple" |
| "Trivial forwarding — single delegation, no branching" | "I'll test after" |
| "Type declaration only — no runtime behavior" | "Already mentally tested it" |
| "Throwaway exploration in tmp/ — not reaching production" | "This is different because..." |
