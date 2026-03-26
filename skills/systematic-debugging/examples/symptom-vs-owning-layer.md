# Symptom Site vs. Owning Layer

This contrast pair illustrates the most common misidentification in failure
diagnosis: treating the file or line where the failure surfaces as the layer
that owns the problem.

---

## Bad Case

**Failure:**
```
AssertionError: expected {"id": 1, "name": "Alice"} to equal {"id": 1, "name": "alice"}
  at test/user.test.ts:42
```

**Diagnosis:**
"The test at `test/user.test.ts:42` is asserting the wrong value. I'll fix the
expectation to use `"Alice"` instead of `"alice"`."

**What went wrong:**

The symptom site is the assertion line in the test file. The diagnosis stopped
there and proposed weakening the expectation.

This skips the real question: why is the name coming back as `"Alice"` when the
test expects `"alice"`? The assertion could be correct — the likely owner is the
normalization layer that should be lowercasing the name before it reaches the
response. Changing the assertion removes the symptom but leaves the bug in place.

---

## Corrected Case

**Same failure.**

**Step 1 — Restate the concrete failing behavior:**
The response returns `"Alice"` (capitalized), but the test expects `"alice"`
(lowercase). The assertion is checking a specific normalization contract.

**Step 2 — Separate symptom site from owning layer:**
Symptom site: `test/user.test.ts:42` (where the mismatch surfaces).
Candidate owning layers: the serialization layer, the normalization step in the
service, or the data layer — not the test itself.

**Step 3 — Check competing explanations:**
- Expectation wrong? Only if there is no normalization contract. Read the test
  setup and nearby tests to check whether lowercasing is an established expectation.
- Implementation wrong? If nearby tests confirm lowercasing is expected, trace
  whether `UserService` or its serializer applies the transformation.
- Fixture wrong? Check whether the fixture stores `"Alice"` and bypasses
  normalization during test setup.

**Step 4 — Owning layer with evidence:**
Nearby tests at lines 50–60 also assert lowercase names, confirming the contract
exists. Reading `UserService.serialize()` shows no lowercasing step. The owning
layer is `UserService.serialize()`, not the test.

**Why this is correct:**

The failure path was traced from assertion → contract confirmation → serializer
inspection. The owning layer is identified with evidence, not just proximity to
the stack trace. The test expectation is preserved.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Symptom site | `test/user.test.ts:42` | `test/user.test.ts:42` |
| Identified owner | The test (the symptom site) | `UserService.serialize()` |
| Competing explanations checked | No | Yes — expectation, implementation, fixture |
| Evidence used | None — proximity only | Nearby tests + serializer inspection |
| Edit proposed | Weaken the assertion | Fix the serializer |
