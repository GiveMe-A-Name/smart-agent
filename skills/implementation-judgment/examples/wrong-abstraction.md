# Wrong Abstraction — Surface Similarity vs. True Duplication

This contrast pair illustrates the judgment call around abstraction: when does
extracting "duplicated" code create a wrong abstraction that costs more than
the duplication it removed?

---

## Scenario

**Request:** "Add rate limiting to the API endpoints. The `/users` endpoint
should limit to 100 req/min per user, and the `/search` endpoint should limit
to 20 req/min per IP."

**Current code shape:**
```typescript
// routes/users.ts
router.get('/users', async (req, res) => {
  const userId = req.auth.userId
  const users = await UserService.list(userId)
  res.json(users)
})

// routes/search.ts
router.get('/search', async (req, res) => {
  const query = req.query.q
  const results = await SearchService.search(query)
  res.json(results)
})
```

No rate limiting exists yet. Both endpoints need it.

---

## Bad Case — Premature Abstraction from Surface Similarity

**Reasoning:** "Both endpoints need rate limiting. I'll extract a shared
rate limiter middleware to avoid duplication."

**Implementation:**
```typescript
// middleware/rateLimiter.ts
function createRateLimiter(options: {
  keyExtractor: (req: Request) => string,
  limit: number,
  window: number,
  onLimitExceeded?: (req: Request) => void,
}) {
  const store = new Map<string, { count: number, resetAt: number }>()
  
  return async (req: Request, res: Response, next: NextFunction) => {
    const key = options.keyExtractor(req)
    const now = Date.now()
    const entry = store.get(key)
    
    if (entry && entry.resetAt > now) {
      if (entry.count >= options.limit) {
        options.onLimitExceeded?.(req)
        return res.status(429).json({ error: 'Rate limited' })
      }
      entry.count++
    } else {
      store.set(key, { count: 1, resetAt: now + options.window })
    }
    next()
  }
}

// routes/users.ts
router.get('/users',
  createRateLimiter({
    keyExtractor: (req) => req.auth.userId,
    limit: 100,
    window: 60000,
  }),
  async (req, res) => { /* ... */ }
)

// routes/search.ts
router.get('/search',
  createRateLimiter({
    keyExtractor: (req) => req.ip,
    limit: 20,
    window: 60000,
    onLimitExceeded: (req) => logSuspiciousActivity(req.ip),
  }),
  async (req, res) => { /* ... */ }
)
```

**Why this looks right but isn't:**

This looks clean. But these two rate limiters are solving different problems
that happen to share the same name:

- The user endpoint limits authenticated users to prevent abuse of their own
  quota. When limits are hit, it's informational.
- The search endpoint limits by IP to defend against scraping and DDoS. When
  limits are hit, it's a security event that needs logging, possible IP
  banning, and escalation.

Within weeks, the search rate limiter needs:
- Sliding window instead of fixed window (to prevent burst-at-boundary attacks)
- IP reputation scoring integration
- Different limits for authenticated vs. anonymous users
- Automatic IP blocking after repeated violations

The user rate limiter needs:
- Per-plan limits (free vs. paid users)
- Graceful degradation (return cached data instead of 429)
- Rate limit headers (X-RateLimit-Remaining, etc.)

Now `createRateLimiter` accumulates parameters and conditionals:

```typescript
function createRateLimiter(options: {
  keyExtractor: (req: Request) => string,
  limit: number | ((req: Request) => number),  // now dynamic
  window: number,
  windowType: 'fixed' | 'sliding',  // new parameter
  onLimitExceeded?: (req: Request) => void,
  returnCachedOnLimit?: boolean,  // new parameter
  addRateLimitHeaders?: boolean,  // new parameter
  autoBlockAfter?: number,  // new parameter
  // ... growing
}) {
```

The abstraction is now harder to understand than two separate implementations
would have been. Every caller must navigate options that only exist for the
other caller's use case. The shared interface became a liability.

---

## Corrected Case — Tolerating Duplication Until Patterns Stabilize

**Step 1 — Recognize surface similarity vs. true duplication:**

Both endpoints "need rate limiting," but the underlying concerns are different:
quota management vs. security defense. They will change for different reasons
at different times.

**Step 2 — Apply the judgment dimension (Abstraction Correctness):**

- Are these cases truly the same, or do they just look the same right now?
  → They look the same right now but are already showing signs of divergence
  (different key strategies, different on-exceed behaviors).
- Would extracting a shared abstraction make both cases harder to change
  independently? → Yes. Each case has its own trajectory.
- Is there a risk of the abstraction accumulating conditionals? → High.
  The diverging requirements are already visible.

**Step 3 — Decide: duplicate or abstract?**

Keep them separate. Write two independent rate limiting implementations,
each tailored to its specific concern.

**Result:**
```typescript
// middleware/userQuotaLimiter.ts
// Owns: per-user quota enforcement for authenticated endpoints
export function userQuotaLimiter() {
  return async (req: Request, res: Response, next: NextFunction) => {
    const userId = req.auth.userId
    const plan = await getUserPlan(userId)
    const usage = await getUsage(userId)
    
    if (usage >= plan.requestLimit) {
      res.set('X-RateLimit-Remaining', '0')
      res.set('X-RateLimit-Reset', plan.resetTime.toString())
      return res.status(429).json({
        error: 'Quota exceeded',
        upgradeUrl: '/pricing'
      })
    }
    
    await incrementUsage(userId)
    res.set('X-RateLimit-Remaining', String(plan.requestLimit - usage - 1))
    next()
  }
}

// middleware/ipDefenseLimiter.ts
// Owns: IP-based defense against scraping and abuse
export function ipDefenseLimiter(limit: number) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const ip = req.ip
    const window = slidingWindow(ip, 60000)
    
    if (window.count >= limit) {
      await logSuspiciousActivity(ip)
      if (window.violations > 3) {
        await blockIp(ip, '1h')
      }
      return res.status(429).json({ error: 'Too many requests' })
    }
    
    window.increment()
    next()
  }
}
```

Each implementation is simple, focused, and easy to change independently.
If a shared pattern genuinely emerges later (e.g., both need the same
sliding window algorithm), that specific utility can be extracted then —
based on actual usage evidence, not predicted similarity.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Rate limiting works initially | Yes | Yes |
| Agent checked if cases are truly the same | No — assumed surface similarity = duplication | Yes — recognized different concerns |
| Abstraction accumulates conditionals over time | Yes — each case pulls it in a different direction | No — each case evolves independently |
| Adding new requirements is easy | No — must navigate shared options | Yes — change one without affecting the other |
| Code is easy to delete/replace | No — shared abstraction entangles both | Yes — each can be replaced independently |

The test is not "do these code blocks look similar?" but "do these code
blocks change for the same reasons at the same time?" If they don't, keeping
them separate is cheaper than the wrong abstraction — even if it means some
lines of code appear in both places.
