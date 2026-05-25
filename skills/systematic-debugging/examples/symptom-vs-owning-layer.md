# Symptom Site vs. Owning Layer

Use this example when a proposed fix targets the stack-trace frame, assertion line, or nearest code before evidence shows that location produced the invalid state.

## Base Case: Stack-Trace Anchoring

**Observed failure:**

```
AssertionError: expected {"id": 1, "name": "Alice"} to equal {"id": 1, "name": "alice"}
  at test/user.test.ts:42
```

**Invalid diagnosis:**

The assertion at `test/user.test.ts:42` differs from the received value, so the test expectation should be changed to `"Alice"`.

**Why the diagnosis fails:**

The assertion line is the symptom site. It proves where the mismatch surfaced, not which layer owns the capitalization behavior. Changing the expected value weakens the contract before checking whether lowercasing is required.

**Corrected diagnosis:**

1. Restate the concrete mismatch: the response returns `"Alice"`, while the assertion expects `"alice"`.
2. Identify the contract being tested: response names should be normalized to lowercase.
3. List candidate owning layers: test expectation, fixture setup, serializer, service normalization, or data source.
4. Check evidence for each plausible candidate before editing.

**Owning layer with evidence:**

Nearby tests also assert lowercase names, so the expectation is a real contract. Reading `UserService.serialize()` shows no lowercasing step. The owning layer is `UserService.serialize()`, not `test/user.test.ts:42`.

| Check | Invalid diagnosis | Corrected diagnosis |
|---|---|---|
| Symptom site | `test/user.test.ts:42` | `test/user.test.ts:42` |
| Ownership claim | Test owns the failure | Serializer owns the missing normalization |
| Evidence used | Stack-trace proximity only | Nearby tests plus serializer inspection |
| Edit | Weaken the assertion | Fix `UserService.serialize()` |

## Variant: Bypass Changes Evidence

Use this variant when context says a fixture, mock, helper, migration, import path, or test setup bypasses a normal layer, but the diagnosis still treats the stack-trace function as the owner.

**Observed failure:**

```
AssertionError: expected 'alice@example.com' to equal 'alice'
  at UserEmailParser.parse (src/utils/email_parser.ts:15)
```

**Observed context:**

- `src/utils/email_parser.ts` parses emails and is used by multiple services.
- `src/services/user.ts` calls the parser on the normal service path.
- `test/fixtures/user_fixture.ts` creates user objects directly and bypasses the service layer.

**Invalid diagnosis:**

The stack trace points to `UserEmailParser.parse`, and the received value is the full email, so `email_parser.ts` owns the failure.

**Why the diagnosis fails:**

The bypass fact changes what the stack trace proves. If fixture setup bypasses `UserService`, then `UserEmailParser.parse()` may not be called on the fixture path. The stack trace can come from the test's direct verification call rather than from the path that created the bad user object.

**Required recovery action:**

Before editing the stack-trace function, check whether that function is reached on the bypass path.

```
Normal path:  UserService -> UserEmailParser.parse() -> user.email = 'alice'
Fixture path: user_fixture.ts -> user.email = 'alice@example.com'
```

**Owning layer with evidence:**

The fixture path skips `UserService`, so it stores raw email values without parser normalization. The owning layer is `test/fixtures/user_fixture.ts`, not `email_parser.ts`. The parser can still be tested separately, but this failure does not prove parser ownership.

| Check | Invalid diagnosis | Corrected diagnosis |
|---|---|---|
| Bypass evidence | Acknowledged but not used | Used to reinterpret the stack trace |
| Stack-trace meaning | Parser is broken | Parser was called during verification, not fixture setup |
| Evidence check | Parser return value only | Reachability on the fixture path |
| Ownership claim | `email_parser.ts` | `test/fixtures/user_fixture.ts` |
| Edit | Fix parser | Fix fixture setup to apply the normal transformation |
