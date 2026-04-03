# Ambiguous Revert Requests

This contrast pair illustrates a specific false-clarity trap: requests like
"restore old behavior / keep original logic / revert to before X" appear to have
a clear referent (the old code) but actually carry hidden branch points when the
current codebase has evolved beyond the referenced baseline.

The failure mode is treating "the old code exists" as proof that copying it back
is the correct action, and planning a strict structural revert without surfacing
the real decision.

---

## Scenario

**Request:** "Revert the auth middleware to how it worked before the v2 refactor —
it's breaking the admin panel."

Repository inspection reveals: the v2 refactor added token refresh logic and
multi-tenant scoping that did not exist in the pre-v2 baseline. The admin panel
uses the token refresh path. A strict structural revert would remove that path.

---

## Bad Case

**Agent response:**

> "I'll restore the pre-v2 auth middleware. I'll find the last commit before the
> refactor and revert the changed files."

**What went wrong:**

The agent treated the request as unambiguous. "Revert to before v2" sounds like
a clear instruction, but it actually has three valid interpretations:

- **(a) Strict structural revert**: copy the pre-v2 code back, discarding token
  refresh and multi-tenant scoping
- **(b) Regression fix only**: keep the v2 capability paths, identify what
  specifically broke the admin panel, fix only that
- **(c) Behavior revert for a specific scenario**: change how the middleware
  behaves for admin-panel requests only, preserving v2 behavior elsewhere

The admin panel uses the token refresh path. A strict revert (a) would break a
different set of things. The user may not realize this tradeoff exists.

---

## Corrected Case

**Step 1 — Inspect before planning:**

Check whether the current codebase has capability paths that postdate the
referenced baseline. Find: token refresh (added in v2), multi-tenant scoping
(added in v2). Confirm: admin panel uses the token refresh path.

**Step 2 — Classify the unknowns:**

| Unknown | Type | Can self-resolve? |
|---|---|---|
| What the v2 refactor changed | repository fact | Yes — git diff or file inspection |
| What specifically is breaking the admin panel | repository fact | Possibly — check error logs or failing paths |
| Whether a strict structural revert is intended | user intent | No — hidden tradeoff |

**Step 3 — Surface the real decision:**

> "The v2 refactor added token refresh and multi-tenant scoping — the admin panel
> currently uses the token refresh path. A strict revert to pre-v2 would remove
> that path and likely break different things.
>
> Before I plan: do you want to (a) fully revert to pre-v2 structure even if
> that removes token refresh, (b) keep token refresh and fix only what's broken
> in the admin panel, or (c) something else?"

**Why this is correct:**

The agent resolved `repository facts` by inspection, then surfaced the hidden
tradeoff before planning. The user is now choosing between real options with real
consequences — not approving a plan they didn't fully understand.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Checked for postdate capability paths | No | Yes |
| Hidden tradeoff surfaced | No | Yes |
| User is making an informed choice | No | Yes |
| Plan starts without confirmation | Yes | No |

---

## When This Applies

Whenever a request uses: "restore", "revert", "go back to how it was", "undo
the X change", "keep the original logic" — check first whether the current
codebase has evolved beyond the referenced state. If it has, the request has at
least two valid interpretations. Name the choice explicitly before planning.

Do not treat the existence of old code as proof that restoring it is the correct
action.
