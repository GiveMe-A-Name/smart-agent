# Abstraction Change Drivers - Separate vs. Consolidate

Read this when similar or repeated code makes extraction tempting, or when a
review asks to "deduplicate" code that may not share one owner.

Good code uses abstraction to preserve the right change boundary. Do not decide
from duplication count or surface similarity. Decide from the event that would
force each caller to change.

---

## Decision Check

Before extracting or consolidating:

- Name each caller and the policy, rule, or lifecycle it owns.
- For each copy, answer: "What future request would require this code to
  change?"
- Consolidate when the same future request should change every copy.
- Keep separate when different future requests should change different callers.
- Extract only the stable lower-level primitive when policies differ but share a
  mechanism.

Risk signal: a shared helper grows options, flags, or conditionals that map
one-to-one to caller names, routes, products, or use cases. That helper is acting
as a dispatcher for unrelated policies.

---

## Case A - Keep Separate When Change Drivers Diverge

**Task:** "Add rate limiting to `/users` and `/search`."

```text
// /users
limit authenticated users to 100 requests/minute per user

// /search
limit anonymous traffic to 20 requests/minute per IP
```

Bad move:

```typescript
createRateLimiter({ keyExtractor: (req) => req.auth.userId, limit: 100 })
createRateLimiter({
  keyExtractor: (req) => req.ip,
  limit: 20,
  onLimitExceeded: (req) => logIp(req.ip),
})
```

Both snippets are "rate limiting," but the change drivers differ:

- `/users` is quota management. Future changes may add per-plan limits, upgrade
  messaging, quota headers, or cached fallback behavior.
- `/search` is abuse defense. Future changes may add IP reputation, escalation,
  sliding windows, or auto-blocking.

Do not bind these policies behind one option-heavy middleware:

```typescript
createRateLimiter({
  keyExtractor,
  limit,
  onLimitExceeded,
  addRateLimitHeaders,
  returnCachedOnLimit,
  autoBlockAfter,
})
```

Correct shape:

```typescript
export function userQuotaLimiter() {}
export function ipDefenseLimiter() {}
```

If both need the same counter, extract only that primitive:

```typescript
export function slidingWindowCounter(key: string, windowMs: number) {}
```

---

## Case B - Consolidate When Change Drivers Match

**Task:** "Address path traversal review comments in CLI env-loading flows."

Three files must enforce the same rule:

```text
commands/create.ts
commands/build.ts
utils/loadEnv.ts
```

Shared rule: user-controlled or config-controlled paths must not escape the
approved root.

Bad move:

```typescript
// commands/create.ts
return relative !== '' && !relative.startsWith('..')

// commands/build.ts
return !relative.startsWith('..')

// utils/loadEnv.ts
return relative === '' || !relative.startsWith('..')
```

Each local helper can satisfy its local review comment, but ownership of one
security rule has been multiplied. Future changes to symlink handling, empty-path
semantics, path normalization, or allowed roots can update one copy and miss the
others.

Correct shape:

```typescript
// utils/pathSafety.ts
export function isPathInside(parent: string, child: string): boolean {
  const relative = path.relative(parent, child)
  return relative === '' || (!relative.startsWith('..') && !path.isAbsolute(relative))
}
```

All call sites use the same helper. If one call site needs different semantics,
name that distinct rule at the boundary instead of hiding it inside a copied
local helper.

---

## Apply the Distinction

Use this table as the action boundary:

| Check | Keep separate | Consolidate |
|---|---|---|
| Future request changes | One caller's policy | Every copy of the same rule |
| Main risk if wrong | Option-heavy abstraction with caller-specific branches | Rule drift across copied helpers |
| Correct action | Separate policy owners; extract only stable primitives | One owner for the shared rule |

Do not extract because two snippets look alike. Do not copy because a helper is
small. Name the change driver first, then choose the code shape.
