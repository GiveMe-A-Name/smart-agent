# TDD Cycle Reference

## Red-Green-Refactor

### RED — Write a Failing Test

Write one minimal test showing what should happen.

**Requirements:**
- One behavior per test
- Name describes the behavior, not the implementation
- Tests real code, not mocks (mocks only when unavoidable)

See `examples/test-quality.md` for a Good/Bad contrast pair.

### Verify RED — Watch It Fail (MANDATORY, never skip)

```bash
npm test path/to/test.test.ts
```

Confirm:
- Test fails, not errors
- Failure message is the expected one
- Fails because the feature is missing, not due to a typo

**Test passes immediately?** You are testing existing behavior. Fix the test.
**Test errors?** Fix the error, re-run until it fails correctly.

### GREEN — Minimal Code

Write the simplest code that passes the test. Do not add features, refactor other code, or "improve" beyond what the test requires.

See `examples/test-quality.md` for a Good/Bad contrast pair.

### Verify GREEN — Watch It Pass (MANDATORY)

```bash
npm test path/to/test.test.ts
```

Confirm:
- Test passes
- Tests related to this change pass
- No new errors or warnings introduced by this change

**Test fails?** Fix code, not the test.
**Related tests fail?** Fix before continuing.

### REFACTOR — Clean Up

After green only:
- Remove duplication
- Improve names
- Extract helpers

Do not add behavior. Keep tests green throughout.

### Repeat

Write the next failing test for the next behavior.

---

## Completion Signals

Work is complete when:

- Every behavior added has a test that was watched fail before the code existed
- Tests related to this change pass
- Test output introduced by this change is clean — warnings or errors that predate the change and fall outside its scope do not need to be resolved
- Tests verify real behavior, not mock behavior or implementation details

If you cannot explain why each test failed before implementing, TDD was not followed.

---

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests written after code pass immediately. Passing immediately proves nothing. |
| "Tests after achieve the same goals" | Tests-after answer "what does this do?" Tests-first answer "what should this do?" |
| "Already manually tested" | Ad-hoc ≠ systematic. No record, can't re-run, easy to forget under pressure. |
| "Deleting X hours of work is wasteful" | Sunk cost. Keeping unverified code is technical debt. |
| "Keep as reference, write tests first" | You will adapt it. That is testing after. Delete means delete. |
| "Need to explore first" | Fine. Throw away the exploration. Start with TDD. |
| "TDD will slow me down" | TDD is faster than debugging. Pragmatic = test-first. |
| "This is different because..." | Stop. That sentence ends in rationalization. |

---

## When Stuck

| Problem | Solution |
|---------|----------|
| Don't know how to test | Write the wished-for API. Write the assertion first. If still stuck, ask. |
| Test too complicated | The design is too complicated. Simplify the interface. |
| Must mock everything | Code is too coupled. Use dependency injection. |
| Test setup is huge | Extract helpers. Still complex? Simplify the design. |

---

## Red Flags — Stop and Start Over

- Code before test
- Test passes immediately
- Cannot explain why the test failed
- Tests added "later"
- "I already manually tested it"
- "Keep as reference" or "adapt existing code"
- "Already spent X hours, deleting is wasteful"
- "TDD is dogmatic, I'm being pragmatic"
- "This is different because..."

All of these mean: delete the code, start over with TDD.
