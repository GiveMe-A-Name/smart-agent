# Fixture Bypass Detection

This contrast pair illustrates a subtler form of stack-trace anchoring: the
model sees explicit bypass information but still blames the stack-trace function
because it appears to have evidence (the function returned a wrong value).

---

## Bad Case

**Failure:**
```
AssertionError: expected 'alice@example.com' to equal 'alice'
  at UserEmailParser.parse (src/utils/email_parser.ts:15)
```

**Context provided:**
- `src/utils/email_parser.ts` — parses emails, used by multiple services
- `src/services/user.ts` — UserService
- `test/fixtures/user_fixture.ts` — creates user objects, bypasses the service layer

**Diagnosis:**
"The failure originates at `src/utils/email_parser.ts:15`. The stack trace points
directly to `UserEmailParser.parse`, which returns the full email instead of
the local part. The fixture bypasses the service layer, but the parser is still
being called and returning the wrong value — so the parser owns this failure."

**What went wrong:**

The bypass information was read but not used. The model noted that the fixture
bypasses the service layer, then set that fact aside because the stack trace
appeared to provide direct evidence: parse() was called and returned the wrong
value. The bypass was treated as background context rather than a fact that
changes what the stack-trace evidence means.

The missing question: **if the fixture bypasses the service layer, is
`UserEmailParser.parse` something the fixture path would even be expected to
call?** If not, the stack trace is coming from the test calling parse directly
— and the real question is why the fixture-created user object has an unparsed
email in the first place.

---

## Corrected Case

**Same failure and context.**

**Step 1 — Restate the concrete failing behavior:**
The test expects `'alice'` (local part only) but received `'alice@example.com'`
(full email). The assertion is at `UserEmailParser.parse:15`.

**Step 2 — Separate symptom site from owning layer:**
Symptom site: `email_parser.ts:15` — where the assertion surfaces.
Before accepting this as the owning layer, check whether `parse()` is even
supposed to be called via the fixture path.

**Step 3 — Check whether the bypass changes the evidence:**
The prompt says the fixture creates user objects directly, bypassing
`UserService`. The normal path would be:

```
UserService → UserEmailParser.parse() → user.email = 'alice'
```

The fixture path skips `UserService`, so `UserEmailParser.parse()` is never
called during fixture setup. The test likely calls `parse()` separately to
verify behavior — but the fixture-created user already carries the unparsed
email `'alice@example.com'` as its email field.

The stack trace is from the test's direct verification call to `parse()`, not
from a broken parser invoked via the normal path. The parser may be working
correctly; the fixture is the layer that failed to apply it.

**Step 4 — Check competing explanations:**
- Parser broken? Possible, but then it would also fail for service-layer users.
  The bypass means we cannot tell from this test alone.
- Fixture sets email without parsing? Consistent with the bypass description and
  the assertion value. This is the primary candidate.
- Expectation wrong? Only if emails are not supposed to be normalized to local
  part — but the test and the service path both suggest they should be.

**Step 5 — Owning layer with evidence:**
The fixture bypasses the service layer, which means it stores raw email values
without calling the parser. The owning layer is `test/fixtures/user_fixture.ts`,
not `email_parser.ts`. The parser code may be entirely correct; it simply is
not invoked by the fixture path.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Bypass information | Noted, then set aside | Used to re-examine what the stack trace means |
| Stack-trace evidence | Treated as proof the function is broken | Treated as evidence to interrogate given the bypass |
| Key question asked | "What does parse() return?" | "Is parse() even called via the fixture path?" |
| Identified owner | `email_parser.ts` | `test/fixtures/user_fixture.ts` |
| Edit proposed | Fix the parser | Fix the fixture to apply parsing when creating objects |
