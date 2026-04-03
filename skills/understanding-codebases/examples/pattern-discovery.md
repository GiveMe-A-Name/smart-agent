# Pattern Discovery Before Implementing

This example shows how to use an existing feature as a map before adding a new one — avoiding both missed extension points and convention violations.

---

## Task

"Add a new webhook event type for user deletion"

**Investigation strategy**: pattern discovery + convention tracing.

---

## Investigation

**Step 1 — Find a similar feature**: Search for an existing webhook event type (e.g., `user.created`). Find it defined in `events/user-created.ts`.

**Step 2 — Discover the pattern**: The event type is registered in `events/index.ts`, has a corresponding schema in `schemas/events/`, a handler in `handlers/webhooks/`, and a test in `tests/events/`.

**Step 3 — Trace the convention**: Events follow a naming pattern, use a base class `WebhookEvent`, register through a central registry, and have integration tests that verify delivery.

---

## Result

To add `user.deleted`, follow the same pattern: create event definition, add schema, register it, add handler, write tests matching existing test structure. The extension point is the event registry, not the webhook dispatcher.

---

## What This Prevents

Implementing the feature in an ad-hoc way that ignores existing conventions; missing the registration step; putting code in the wrong layer.
