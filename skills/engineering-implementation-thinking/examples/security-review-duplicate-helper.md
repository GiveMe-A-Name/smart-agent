# Security Review Fix - Shared Helper vs. Scattered Patch

This contrast pair illustrates a common failure mode during review-driven fixes:
the patch may address the immediate issue, but still lower engineering quality if
the underlying shared logic remains scattered.

---

## Scenario

**Request:** "Address the medium/high-risk path traversal review comments in the
CLI env-loading flow."

**Current code shape:**

Three touched files now need to validate whether a resolved path stays inside an
allowed directory:

```
// commands/create.ts
// commands/build.ts
// utils/loadEnv.ts
```

The review feedback is about the same safety boundary in all three places:
user-controlled or config-controlled paths must not escape the intended root.

---

## Bad Case - Local Fixes That Quietly Spread Duplication

**Implementation:**

Add a separate `isSubPath` helper to each file:

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

Each call site now has a local check. The review comments can be marked fixed.

**Why this is wrong:**

The patch looks efficient, but it has multiplied ownership of the same security
decision. The review did not reveal three separate implementation problems. It
revealed one shared boundary rule that now has three drifting copies.

Even if the helpers start identical, they are likely to diverge the next time
someone adjusts symlink handling, empty-relative-path semantics, normalization,
or fallback behavior. A future fix may update one location and miss the others,
which is especially dangerous for security-sensitive logic.

You cannot judge this patch only by whether each call site now "has a check."
You also have to ask whether the abstraction has landed in the right owning
location and whether repetition is being reduced or allowed to spread. That is
the engineering quality difference.

The agent treated review remediation as permission to make several local fixes.
It did not stop to ask whether the repeated helper itself was evidence that the
responsibility had not been given a coherent home.

---

## Corrected Case - Fix The Review By Re-Centering Ownership

**Step 1 - Read the review as a boundary signal:**

The review comments across `create.ts`, `build.ts`, and `loadEnv.ts` all point
to the same path-safety rule. That is evidence of shared logic, not three fully
independent behaviors.

**Step 2 - Ask the Core Questions:**

- Why is the same safety check needed in multiple files? Because several flows
  depend on the same boundary: resolved paths must stay inside an approved root.
- Would local helpers keep responsibilities clear? No - they would duplicate a
  security decision that should stay consistent.
- Does a narrow patch make future work harder? Yes - every rule change would
  require finding and updating several copies.

**Step 3 - Decide: narrow patch or focused structural improvement?**

This is not a cue for a broad refactor. The right move is a focused structural
improvement: introduce one shared helper in the owning path-safety module and
route all three call sites through it.

**Result:**

```typescript
// utils/pathSafety.ts
export function isPathInside(parent: string, child: string): boolean {
  const relative = path.relative(parent, child)
  return relative === '' || (!relative.startsWith('..') && !path.isAbsolute(relative))
}
```

```typescript
// commands/create.ts
import { isPathInside } from '../utils/pathSafety'
```

```typescript
// commands/build.ts
import { isPathInside } from '../utils/pathSafety'
```

```typescript
// utils/loadEnv.ts
import { isPathInside } from './pathSafety'
```

If one call site also needs a slightly different behavior, that difference
should be made explicit at the shared boundary rather than hidden inside one of
several copy-pasted helpers.

The review is fixed, but the implementation also leaves the codebase in a more
coherent state: one ownership point, one security semantic, and one place to
change when the rule evolves.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Review comments are addressed | Yes | Yes |
| Shared security logic has one owner | No - duplicated per file | Yes - centralized helper |
| Repetition is shrinking or expanding | Expanding | Shrinking |
| Future rule changes stay coherent | No - drift risk across copies | Yes - one update point |
| Scope of improvement | Superficially small but structurally messy | Focused and bounded |

The test is not only whether the review issue was fixed. It is whether the fix
helped the abstraction return to the right place and stopped the repeated logic
from spreading further.
