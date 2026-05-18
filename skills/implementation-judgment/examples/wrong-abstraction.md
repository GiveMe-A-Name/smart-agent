# Abstraction Change Drivers - Separate vs. Consolidate

This contrast pair calibrates both sides of abstraction judgment:

- surface-similar code should stay separate when callers change for different reasons
- repeated logic should be consolidated when callers share the same safety rule, business rule, or lifecycle driver

The question is not "how many copies exist?" or "do these snippets look similar?"
The question is: "What event would require each copy to change?"

---

## Case A - Keep Separate When Change Drivers Diverge

**Task:** "Add rate limiting to `/users` and `/search`."

**Current need:**

- `/users`: limit authenticated users to 100 requests/minute per user
- `/search`: limit anonymous traffic to 20 requests/minute per IP

**Bad move:** create one generic `createRateLimiter(options)` because both endpoints
"need rate limiting."

```typescript
createRateLimiter({
  keyExtractor: (req) => req.auth.userId,
  limit: 100,
})

createRateLimiter({
  keyExtractor: (req) => req.ip,
  limit: 20,
  onLimitExceeded: (req) => logSuspiciousActivity(req.ip),
})
```

This looks clean, but the two callers change for different reasons:

- The user limiter is quota management. It may need per-plan limits, rate-limit
  headers, upgrade messaging, or cached fallback behavior.
- The search limiter is abuse defense. It may need IP reputation, sliding windows,
  escalation, or automatic blocking.

The shared abstraction will accumulate caller-specific options:

```typescript
createRateLimiter({
  keyExtractor,
  limit,
  windowType,
  onLimitExceeded,
  returnCachedOnLimit,
  addRateLimitHeaders,
  autoBlockAfter,
})
```

**Corrected move:** keep the concerns separate.

```typescript
// Owns per-user quota enforcement.
export function userQuotaLimiter() {}

// Owns IP-based abuse defense.
export function ipDefenseLimiter() {}
```

If both later need the same sliding-window primitive, extract that primitive only.
Do not bind quota management and abuse defense behind one option-heavy middleware.

---

## Case B - Consolidate When Change Drivers Match

**Task:** "Address path traversal review comments in CLI env-loading flows."

**Current need:**

Three files need to enforce the same rule:

```text
commands/create.ts
commands/build.ts
utils/loadEnv.ts
```

The shared rule is: user-controlled or config-controlled paths must not escape the
approved root.

**Bad move:** add a local helper to each file.

```typescript
// commands/create.ts
function isSubPath(parent: string, child: string): boolean {
  const relative = path.relative(parent, child)
  return relative !== '' && !relative.startsWith('..') && !path.isAbsolute(relative)
}
```

```typescript
// commands/build.ts
function isSubPath(parent: string, child: string): boolean {
  const relative = path.relative(parent, child)
  return !relative.startsWith('..') && !path.isAbsolute(relative)
}
```

```typescript
// utils/loadEnv.ts
function isSubPath(parent: string, child: string): boolean {
  const relative = path.relative(parent, child)
  return relative === '' || (!relative.startsWith('..') && !path.isAbsolute(relative))
}
```

Each patch can satisfy its local review comment, but ownership of one security
decision has been multiplied. Future changes to symlink handling, empty-path
semantics, or normalization may update one copy and miss the others.

**Corrected move:** consolidate the shared boundary rule in one owner.

```typescript
// utils/pathSafety.ts
export function isPathInside(parent: string, child: string): boolean {
  const relative = path.relative(parent, child)
  return relative === '' || (!relative.startsWith('..') && !path.isAbsolute(relative))
}
```

Then all three call sites use the same helper. If one call site needs a different
rule, make that difference explicit at the shared boundary rather than hiding it
inside a copied local helper.

---

## The Distinction

| Question | Keep separate | Consolidate |
|---|---|---|
| Do callers change for the same reason? | No | Yes |
| What would a future requirement affect? | One concern independently | Every copy of the same rule |
| Main risk if abstracted/consolidated wrong | Option-heavy abstraction with caller-specific branches | Security or business rule drift across copies |
| Correct move | Separate implementations; extract only stable primitives | One owner for the shared rule |

Use duplication count as a signal, not a verdict. Two copies with different
change drivers may need separation; three copies of one security rule may need
one owner immediately.
