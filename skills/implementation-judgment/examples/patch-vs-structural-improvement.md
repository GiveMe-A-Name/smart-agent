# Narrow Patch vs. Focused Structural Improvement

This contrast pair illustrates the key judgment call in this skill: when does
the "smallest change" become the wrong change? Both cases start from the same
request but reach different implementation decisions.

---

## Scenario

**Request:** "Add email validation to the registration endpoint."

**Current code shape:**
```
// routes/auth.ts
router.post('/register', async (req, res) => {
  const { username, email, password } = req.body
  if (!username) return res.status(400).json({ error: 'username required' })
  if (!password) return res.status(400).json({ error: 'password required' })
  await UserService.create({ username, email, password })
  res.status(201).json({ ok: true })
})
```

The route already validates `username` and `password` inline. `UserService.create`
does no input validation of its own.

---

## Bad Case — Narrow Patch Without Judgment

**Implementation:**
```typescript
if (!email || !email.includes('@')) {
  return res.status(400).json({ error: 'valid email required' })
}
```

Added inline alongside the existing username/password checks.

**Why this is wrong:**

The patch passes. But it has deepened a structural problem that was already
present: input validation logic is scattered inline in the route handler, with
no single ownership point.

When the next field needs validation — or when `UserService.create` is called
from a second route — either the validation gets duplicated there too, or it
gets silently skipped. The patch completed the task while making the next
related change harder.

The agent did not ask: *"Is this inline pattern something that was established
intentionally, or is it an early-stage shortcut that this change should not
extend?"* It treated the current shape as the right model and patched into it.

---

## Corrected Case — Judgment Before Patching

**Step 1 — Read the current shape as evidence:**
Username and password validation are inline in the route. `UserService.create`
does no validation. This is either an early-stage shortcut or a deliberate
choice to keep validation at the HTTP boundary.

**Step 2 — Ask the Core Questions:**
- Why is validation inline here and not in the service? No evidence of
  intentional design — more likely early-stage accumulation.
- Would adding email validation inline duplicate a pattern that should stay
  unified? Yes — if another route calls `UserService.create`, it would need the
  same checks.
- Does the smallest patch make future related changes harder? Yes — it extends
  a pattern that already causes duplication risk.

**Step 3 — Decide: narrow patch or focused structural improvement?**
The existing inline pattern is not protecting a contract or compatibility
boundary. It is an accidental structure that this change has the opportunity
to correct without broadening the work significantly.

A focused structural improvement is warranted: introduce a `validateRegistrationInput`
helper (or move validation into `UserService.create`) so that all registration
input validation has a single ownership point. Scope this tightly — do not
redesign the whole auth module.

**Result:**
```typescript
// services/UserService.ts
static validateRegistrationInput({ username, email, password }) {
  if (!username) throw new ValidationError('username required')
  if (!password) throw new ValidationError('password required')
  if (!email || !email.includes('@')) throw new ValidationError('valid email required')
}

static async create(input) {
  this.validateRegistrationInput(input)
  // ...
}
```

The route handler becomes thin. Future fields validate in one place.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Patch compiles and tests pass | Yes | Yes |
| Agent read the current shape as evidence | No — copied the pattern | Yes — questioned why it exists |
| Agent checked whether patch extends a structural problem | No | Yes |
| Next related change is harder after the patch | Yes — duplication risk grows | No — single validation point |
| Scope of improvement | Unbounded (copied accidental pattern) | Tightly scoped to validation ownership |

The test for the corrected case is not "is the improvement imaginable?" but
"does this specific change expose a structural weakness that a narrow patch
would deepen?" If yes, a focused improvement scoped to that weakness is
warranted.
