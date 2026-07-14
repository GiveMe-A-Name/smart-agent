# Pattern Discovery

## Task

“Add a webhook event type for user deletion.”

## Investigation

Trace the existing `user.created` event rather than starting at the dispatcher. Its definition extends `WebhookEvent`, its schema lives under `schemas/events/`, `events/index.ts` registers it, and integration tests load it through that registry. The dispatcher consumes registered events and contains no per-event policy.

## Working model

- **Current design:** event definitions and schemas are separate from delivery; a central registry connects them to the dispatcher.
- **Runtime path:** registered event → registry-backed dispatcher → webhook handler → delivery.
- **Extension surface:** files are not auto-discovered; the registry is the mechanism that makes an event reachable.
- **Contracts:** the schema defines the payload boundary; `WebhookEvent` defines the event interface.
- **Convention evidence:** the analogue includes a definition, schema, registration, handler, and integration test. Further examples determine whether this is universal or only one instance.
- **Verification surface:** integration tests exercise registry discovery and delivery.
- **Intent:** central discovery may be documented in design material or inferred from repeated wiring; record which.

This evidence explains how existing events participate in the system. Whether the new event should follow, extend, or revise that structure remains an implementation-judgment decision.

## Failure pattern

Editing the dispatcher after searching only for “webhook” misses the registry and schema contracts. One relevant file is not evidence of the system’s extension mechanism.
