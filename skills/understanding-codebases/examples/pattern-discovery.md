# Pattern Discovery Before Implementing

This example shows how to use an existing feature as a map before adding a new one — avoiding both missed extension points and convention violations. The contrast pair shows what gets missed when pattern discovery is skipped.

---

## Task

"Add a new webhook event type for user deletion"

**Investigation strategy**: pattern discovery + convention tracing.

---

## Investigation

**Step 1 — Find a similar feature**: Search for an existing webhook event type (e.g., `user.created`). Find it defined in `events/user-created.ts`.

**Step 2 — Discover the pattern**: The event type is registered in `events/index.ts`, has a corresponding schema in `schemas/events/`, a handler in `handlers/webhooks/`, and a test in `tests/events/`.

**Step 3 — Trace the convention**: Events follow a naming pattern, use a base class `WebhookEvent`, register through a central registry, and have integration tests that verify delivery.

**Step 4 — Check data and discovery gates**: The existing event schema defines payload fields, the registry is imported by the webhook dispatcher, and tests load events through the registry rather than instantiating handlers directly.

---

## Working Model

- **Analogue**: `user.created` shows the existing webhook-event pattern.
- **Convention**: event definition, schema, registry entry, handler, and integration test are all part of the feature shape.
- **Extension point**: the event registry in `events/index.ts`, not the webhook dispatcher.
- **Data path**: the event payload shape belongs in `schemas/events/` and reaches handlers through the registry-backed dispatcher.
- **Discovery gate**: event tests and dispatch load registered events from `events/index.ts`, so adding a handler file alone is not discoverable.
- **Verification path**: write tests matching the existing `tests/events/` integration-test structure.

To add `user.deleted`, follow the same pattern: create event definition, add schema, register it, add handler, and write tests matching existing test structure.

---

## What This Prevents

Implementing the feature in an ad-hoc way that ignores existing conventions; missing the registration step; putting code in the wrong layer.

---

## Bad Case: Skipping Pattern Discovery

**Task:** "Add a new webhook event type for user deletion"

**Step 1:** Search for "webhook" in the codebase. Find `handlers/webhooks/dispatcher.ts`.

**Step 2:** Read `handlers/webhooks/dispatcher.ts`. It dispatches events to registered handlers.

**Step 3:** Add a `user.deleted` case directly to the dispatcher's switch statement. Create a new handler file `handlers/webhooks/user-deleted.ts` following a rough guess at the structure.

**What went wrong:**

The agent never searched for an existing event type to understand the convention. It found one relevant file (the dispatcher) and assumed the dispatcher was the right extension point. It missed:
- the event registry in `events/index.ts` — the actual registration mechanism
- the schema in `schemas/events/` — new events need a schema entry
- the base class `WebhookEvent` — new events extend it in the existing pattern
- the test pattern in `tests/events/` — integration tests verify delivery

The result passes a surface check ("a handler exists") but breaks the convention and will likely fail integration tests.

**The distinction:**

Pattern discovery asks "how does the codebase already do this?" before writing any code. Skipping it means the implementation is guided by guesses about structure rather than evidence of convention. The cost compounds: every downstream file that depends on event registration will also be wrong.

| | Bad case | Corrected case |
|---|---|---|
| First move | Find the dispatcher, add code | Find an existing event type, read it |
| Extension point found | Dispatcher switch statement (wrong) | Event registry in `events/index.ts` (correct) |
| Missed artifacts | Schema, base class, test pattern | None |
| Result | Structurally incomplete, likely broken | Convention-consistent, testable |
