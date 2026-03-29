# Implicit Contract Break — When "Safe" Refactoring Silently Changes Behavior

This contrast pair illustrates a subtle failure mode: a refactoring that
changes no function signatures, passes all existing tests, yet breaks callers
who depended on implicit behavioral properties of the old implementation.

---

## Scenario

**Request:** "Refactor `getConfig` to use async file reading instead of sync,
for performance."

**Current code shape:**
```typescript
// config/loader.ts
let cachedConfig: Config | null = null

export function getConfig(): Config {
  if (cachedConfig) return cachedConfig
  const raw = fs.readFileSync('./config.json', 'utf-8')
  cachedConfig = JSON.parse(raw)
  return cachedConfig
}
```

**Callers in the codebase:**
```typescript
// middleware/auth.ts
function authMiddleware(req, res, next) {
  const config = getConfig()
  const secret = config.jwt.secret
  // ... verifies JWT
}

// startup/init.ts
function initializeApp() {
  const config = getConfig()
  // Called multiple times during startup; relies on getting
  // the same object reference each time
  setupDatabase(config)
  setupCache(config)
  setupLogging(config)
}

// utils/featureFlags.ts
function isFeatureEnabled(name: string): boolean {
  const config = getConfig()
  return config.features?.[name] === true
}
```

---

## Bad Case — Change Signature, Preserve Name, Break Behavior

**Reasoning:** "I'll make it async for better performance. The interface is
basically the same — just add `await` at call sites."

**Implementation:**
```typescript
// config/loader.ts
let cachedConfig: Config | null = null

export async function getConfig(): Promise<Config> {
  if (cachedConfig) return cachedConfig
  const raw = await fs.promises.readFile('./config.json', 'utf-8')
  cachedConfig = JSON.parse(raw)
  return cachedConfig
}
```

**What broke and why:**

The function signature changed (returns `Promise<Config>` instead of `Config`),
so TypeScript catches most call sites. But the deeper damage is behavioral:

1. **Race condition on first load.** The old implementation was synchronous,
   meaning `cachedConfig` was populated before any caller could proceed. The
   new async version has a window where multiple concurrent callers can trigger
   parallel file reads before the cache is populated. Each read may return a
   different `Config` object reference.

2. **`initializeApp` depended on reference identity.** It called `getConfig()`
   three times expecting the same object. With the race condition, the first
   two calls may get one object, and the third gets a different one. If any
   setup step mutates the config object (adds computed defaults, for example),
   those mutations are now inconsistent.

3. **`isFeatureEnabled` was called synchronously in template rendering.**
   Making it async means the entire template rendering pipeline needs to
   become async, or the feature flag is always `undefined` because the
   promise isn't awaited.

4. **Error semantics changed.** The old version threw a synchronous error if
   the file was missing — caught by a `try/catch` around startup. The new
   version returns a rejected promise, which isn't caught by the same
   synchronous `try/catch`, resulting in an unhandled promise rejection that
   kills the process silently instead of showing the startup error message.

The agent focused on the explicit contract (function signature) and missed
the implicit contracts: synchronous execution guarantee, reference identity
on cache hits, synchronous error propagation, and call-site context
assumptions.

---

## Corrected Case — Map Implicit Contracts Before Changing

**Step 1 — Identify implicit contracts by reading callers:**

Before changing the implementation, trace every caller and ask: "what
properties of the current behavior does this caller depend on, beyond the
type signature?"

- `authMiddleware`: calls synchronously in a middleware chain. Expects
  immediate result. Switching to async requires all downstream middleware
  to also be async.
- `initializeApp`: calls multiple times, may depend on reference identity.
  All calls are in a synchronous startup sequence.
- `isFeatureEnabled`: used in synchronous template rendering. Cannot become
  async without a cascade of changes.

**Step 2 — Apply the judgment dimension (Contract and Compatibility):**

- Callers depend on observed behavior (synchronous return, same reference),
  not just the type signature.
- The "sync → async" change is not a drop-in replacement. It changes the
  execution model at every call site.
- The performance benefit of async file I/O for a cached config that is read
  once at startup is negligible — the file is read once, then served from cache.

**Step 3 — Decide: is this refactoring worth the behavioral disruption?**

The cost of making `getConfig` async is high (cascade of changes through many
callers) and the benefit is low (one file read at startup). The right judgment
is either:

**(A) Preserve synchronous behavior, solve the real problem differently:**

If the concern is "blocking the event loop on startup," a targeted fix is to
do the file read before the event loop starts (at module load time) or use
a pre-warmed cache:

```typescript
// config/loader.ts — synchronous behavior preserved, cache warmed eagerly
const cachedConfig: Config = JSON.parse(
  fs.readFileSync('./config.json', 'utf-8')
)

export function getConfig(): Config {
  return cachedConfig
}
```

This is simpler, preserves all implicit contracts, and eliminates the
"first call" performance concern entirely.

**(B) If async is truly needed, make it a new function with a new name:**

```typescript
// config/loader.ts
// Existing sync API preserved for current callers
export function getConfig(): Config {
  if (cachedConfig) return cachedConfig
  const raw = fs.readFileSync('./config.json', 'utf-8')
  cachedConfig = JSON.parse(raw)
  return cachedConfig
}

// New async API for new callers that can handle async
export async function loadConfig(): Promise<Config> {
  if (cachedConfig) return cachedConfig
  const raw = await fs.promises.readFile('./config.json', 'utf-8')
  cachedConfig = JSON.parse(raw)
  return cachedConfig
}
```

The new name signals a new behavioral contract. Existing callers are
unaffected. New callers can opt into the async version. Migration can happen
caller-by-caller when each caller is ready.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Performance concern addressed | Partially | Yes (option A) or Yes with migration path (option B) |
| Existing callers break | Yes — async cascade, race conditions, error handling changes | No — implicit contracts preserved |
| Agent identified implicit contracts before changing | No — only considered the type signature | Yes — traced callers and their behavioral assumptions |
| Scope of change is proportional to benefit | No — massive cascade for negligible benefit | Yes — targeted fix or additive new API |
| Rollback is easy if problems arise | No — async changes cascade everywhere | Yes — old code unchanged |

The key failure was treating "refactoring" as meaning "same function, new
implementation." True behavioral preservation requires understanding what
callers actually depend on — including execution model (sync vs. async),
reference identity, error propagation style, and call-site context. These are
all implicit contracts that the type system does not enforce but callers
absolutely rely on.
