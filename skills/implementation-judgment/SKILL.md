---
name: implementation-judgment
description: "Apply structural engineering judgment before behavior-affecting code. TRIGGER when about to implement a bug fix, feature, patch, refactor, or accepted review feedback that can change runtime behavior, contracts, state, dependencies, tests, or performance. DO NOT TRIGGER for confirmed non-behavioral edits such as formatting, comments, file moves, or exact renames."
---

# Implementation Judgment

Choose how a code change should land before writing behavior-affecting code. The goal is not to make the requested diff smaller or larger by default; it is to avoid moving behavior into the wrong layer, breaking implicit contracts, adding coupling, or making verification pass by weakening what it proves.

## Principles

### Invocation And Evidence

- **Treat the request as intent, not design.** The request names the desired outcome; code evidence determines the responsible layer, owner, and contract to preserve [because user wording, review suggestions, and issue text often point at symptoms rather than the boundary that owns the behavior].
- **Choose a change location only after tracing ownership.** Before writing code, identify the behavior-changing function or module, affected callers or public entry points, and the module boundary that currently owns the behavior; if affected behavior, callers, or acceptance criteria are unknown, gather evidence or clarify intent before choosing the implementation shape.
- **Surface unresolved structural conflicts before coding.** If ownership is unclear, caller contracts would silently change, migration or rollback order is unknown, or multiple dimensions point in different directions, pause and surface the conflict before producing code [because once code exists, pressure shifts toward salvaging it instead of resolving the design choice].

### Ownership And Contracts

- **Keep the smallest change that is structurally honest.** A narrow patch is correct when it preserves ownership, contracts, dependency direction, and future changeability; a tiny patch that deepens a bad boundary is accumulated damage, not conservatism.
- **Put each responsibility where the owner has full context.** Validation, recovery, state mutation, domain rules, and security checks should not be split across layers unless each layer owns a distinct, named concern [because partial ownership creates inconsistent behavior and makes future changes require tracing the whole stack].
- **Assign recovery to the layer that can decide.** Inner layers may classify or propagate errors, but recovery belongs to the caller with enough context to choose retry, user response, rollback, or alerting.
- **Preserve implicit contracts unless the task explicitly changes them.** Callers depend on observed behavior such as return defaults, return shape, thrown errors, side effects, persistence behavior, event emission, sync/async execution, ordering, caching, and reference identity; surface contract changes before implementation unless the user explicitly requested them.

### Boundaries And Abstractions

- **Keep state and dependency direction visible when crossing boundaries.** Before moving code across a module/package boundary, name the state the code reads or writes and the new dependency direction; do not move behavior when the move creates a dependency from stable infrastructure into volatile business logic or makes state ownership ambiguous.
- **Let change drivers decide abstraction.** Extract shared code only when at least two current callers change for the same reason; keep surface-similar code separate when the cases have different product, security, lifecycle, or ownership drivers [because the wrong abstraction couples future changes that should remain independent].
- **Do not hide caller-specific behavior behind switches.** Before adding a parameter, option, or conditional to an existing abstraction, map each branch to its caller; if branches are caller-specific or encode different change drivers, split the abstraction or surface the boundary problem instead of adding the switch.

### Feedback And External Dependencies

- **Implement reviewer concerns, not literal patch locations.** For accepted review feedback, identify the reviewer's concern separately from their suggested code location, then implement the concern in the responsible layer; surface the gap before coding if the feedback implies a goal not confirmed by the user.
- **Verify external contracts before coding against them.** If the implementation depends on an external API, library behavior, generated interface, CLI flag, or function signature not already verified in this session, verify it from local source, installed package types, or official documentation before coding.

### Verification And Scope

- **Treat verification as evidence, not a target.** A passing test suite supports the change only when tests still assert the intended behavior; do not modify tests, snapshots, type suppressions, lint configuration, or verification scope merely to make checks pass.
- **Keep the diff scoped to the implementation decision.** Do not include adjacent cleanup, opportunistic renames, broad refactors, or unrelated style fixes. Note them separately unless they are required to keep the current change structurally coherent.
- **Choose the smallest structurally honest intervention.** Patch locally when ownership, contracts, and dependency direction are clear; make a focused structural improvement only when the current change would otherwise deepen a known boundary problem; pause when ownership, caller contracts, migration order, or dependency direction cannot be determined from evidence.

### Runtime And Performance

- **Treat runtime cost as part of implementation shape.** If a change adds a query, HTTP call, subprocess call, or file read inside a loop, confirm a batch alternative exists or that batching is not applicable.
- **Keep production-scale data bounded.** If a collection is loaded entirely into memory or a hot path adds nested loops over production-sized data, confirm the size is bounded or switch to streaming, pagination, or a different algorithm.
- **Bound external and persistent resources.** New external calls need a timeout or the codebase's timeout wrapper; persistent mutable state in a server, worker, daemon, event consumer, or browser session needs a bound, eviction, rotation, or reset path.
- **Parallelize independent I/O when order does not matter.** Sequential independent I/O calls should run concurrently unless ordering or data dependency prevents it.

See `references/judgment-dimensions.md` when a change touches abstraction, complexity placement, failure handling, contracts, dependencies, observability, security, state, or migration.
See `references/performance-dimensions.md` when performance, resource, frontend, database, async, or concurrency impact is part of the change.

Example index:

| Example | Read when |
|---------|-----------|
| `examples/wrong-abstraction.md` | Repeated or similar code may need separation or consolidation based on change drivers. |
| `examples/complexity-misplacement.md` | Validation, transformation, or recovery logic is being split across layers. |
| `examples/implicit-contract-break.md` | A refactor appears signature-safe but may change observed caller behavior. |
