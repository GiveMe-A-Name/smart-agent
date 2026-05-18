---
name: implementation-judgment
description: "Apply structural engineering judgment before behavior-affecting code. TRIGGER when about to implement a bug fix, feature, patch, refactor, or accepted review feedback that can change runtime behavior, contracts, state, dependencies, tests, or performance. DO NOT TRIGGER for confirmed non-behavioral edits such as formatting, comments, file moves, or exact renames."
---

# Implementation Judgment

Choose how a code change should land before writing behavior-affecting code. The goal is not to make the requested diff smaller or larger by default; it is to avoid moving behavior into the wrong layer, breaking implicit contracts, adding coupling, or making verification pass by weakening what it proves.

## Principles

- **Treat the request as intent, not design.** The request names the desired outcome; code evidence determines the responsible layer, owner, and contract to preserve [because user wording, review suggestions, and issue text often point at symptoms rather than the boundary that owns the behavior].
- **Keep the smallest change that is structurally honest.** A narrow patch is correct when it preserves ownership, contracts, dependency direction, and future changeability; a tiny patch that deepens a bad boundary is accumulated damage, not conservatism.
- **Put each responsibility where the owner has full context.** Validation, recovery, state mutation, domain rules, and security checks should not be split across layers unless each layer owns a distinct, named concern [because partial ownership creates inconsistent behavior and makes future changes require tracing the whole stack].
- **Preserve implicit contracts unless the task explicitly changes them.** Callers depend on observed behavior such as return defaults, error types, side effects, sync/async execution, ordering, caching, and reference identity; signatures and docs do not capture the full contract.
- **Let change drivers decide abstraction.** Extract shared code only when current callers change for the same reason; keep surface-similar code separate when the cases have different product, security, lifecycle, or ownership drivers [because the wrong abstraction couples future changes that should remain independent].
- **Treat verification as evidence, not a target.** A passing test suite supports the change only when the tests still assert the intended behavior; changing tests, snapshots, or type checks to match the new output can make the score green while preserving the bug.

## Constraints

- Apply this skill only after the relevant code evidence and intent needed for the change are available. If the affected behavior, callers, or acceptance criteria are not yet known, gather evidence or clarify intent before choosing an implementation shape.
- Skip this skill only when the edit has confirmed zero behavioral effect from observable evidence: formatting, comments, exact rename, or file move with no import/runtime behavior change. Do not skip because the request is framed as "simple", "quick", or "one-liner" [because simplicity framing is priority information, not structural evidence].
- Before writing code, identify the behavior-changing function or module, the affected callers or public entry points, and the module boundary that currently owns the behavior. If any of these has not been read or traced, do not choose a change location yet.
- If the change alters a default value, return shape, thrown error, side effect, persistence behavior, event emission, ordering, cache semantics, or sync/async execution, treat it as a contract change and surface that before implementation unless the user explicitly requested that contract change.
- Before moving code across a module/package boundary, enumerate the state the code reads or writes and the new dependency direction. Stop if the move creates a dependency from stable infrastructure into volatile business logic, or if state ownership becomes ambiguous.
- Before adding error handling, name the recovery owner. Inner layers may classify or propagate errors, but recovery belongs to the caller with enough context to decide retry, user response, rollback, or alerting.
- Before introducing a shared abstraction, name at least two current callers and the shared change driver. A pure domain-neutral utility may be extracted with two callers when the utility behavior is stable and already conventional in the codebase; speculative interfaces, options objects, and "future use" abstractions do not qualify.
- Before adding a parameter, option, or conditional to an existing abstraction, map each branch to its caller. If branches are caller-specific or encode different change drivers, split the abstraction or surface the boundary problem instead of adding the switch.
- If accepted review feedback is being implemented, identify the reviewer's concern separately from their suggested code location. Implement the concern in the responsible layer; if the feedback implies a goal not confirmed by the user, surface that gap before coding.
- If the implementation depends on an external API, library behavior, generated interface, CLI flag, or function signature not already verified in this session, verify it from local source, installed package types, or official documentation before coding.
- Do not modify tests, snapshots, type suppressions, lint configuration, or verification scope merely to make checks pass. Change verification only when the intended behavior changed or the existing assertion is proven wrong from requirements or code evidence.
- Do not include adjacent cleanup, opportunistic renames, broad refactors, or unrelated style fixes in the implementation. Note them separately unless they are required to keep the current change structurally coherent.

## Performance And Runtime Checks

Check these red lines before accepting the implementation shape:

- Query, HTTP call, subprocess call, or file read inside a loop: confirm a batch alternative exists or is not applicable.
- Collection loaded entirely into memory: confirm the production-scale size is bounded or use streaming/pagination.
- New external call: set a timeout or use the codebase's existing timeout wrapper.
- Hot path with nested loops over production-sized data: confirm the size is bounded or change the algorithm.
- Sequential independent I/O calls: parallelize or state why ordering/data dependency prevents it.
- Persistent mutable state in a server, worker, daemon, event consumer, or browser session: confirm a bound, eviction, rotation, or reset path.

## Scope Decision

- Use a narrow local patch when the existing owner is clear, contracts are preserved, dependencies stay in the same direction, no shared state boundary moves, and no red-line performance/runtime check fires.
- Make a focused structural improvement when the narrow patch would deepen one identified structural problem, such as scattering one concern across layers, duplicating security logic with the same change driver, or adding another caller-specific branch to a strained abstraction.
- Stop and surface the conflict when ownership is unclear, caller contracts would silently change, migration/rollback order is unknown, or multiple dimensions point in different directions. Do not produce code and mention the concern in the same response [because once code exists, pressure shifts toward salvaging it instead of resolving the design choice].

## Evidence Output

When this skill affects the implementation, keep the final explanation short but name the evidence that made the chosen shape sound:

- responsible owner or layer used
- caller/contract impact preserved or surfaced
- state or dependency boundary checked when relevant
- performance/runtime red lines checked when relevant
- verification run, or why verification could not be run

## Stop And Revise

Stop before coding or revise the diff if any condition is true:

- The first editable location was treated as the right location without checking callers and ownership.
- The implementation changes behavior under an existing name without surfacing the contract change.
- A shared abstraction was introduced without current callers that share a change driver.
- A parameter or conditional was added to an abstraction before mapping branches to callers.
- A test, snapshot, type suppression, or linter rule was changed because it was easier than fixing or explaining the underlying behavior.
- The diff removes a currently reachable capability path, fallback, migration behavior, native path, or version-specific branch without surfacing the capability loss first.
- The implementation adds persistent mutable state in a long-running process without a bound or eviction mechanism.
- The task is review-feedback implementation and the literal suggested patch moves responsibility, changes contract, or reverses dependency direction.

---

See `references/judgment-dimensions.md` when a change touches abstraction, complexity placement, failure handling, contracts, dependencies, observability, security, state, or migration.
See `references/risk-triage.md` when deciding between a narrow patch, focused structural improvement, or pause.
See `references/performance-dimensions.md` when performance, resource, frontend, database, async, or concurrency impact is part of the change.

Example index:

| Example | Read when |
|---------|-----------|
| `examples/wrong-abstraction.md` | Repeated or similar code may need separation or consolidation based on change drivers. |
| `examples/complexity-misplacement.md` | Validation, transformation, or recovery logic is being split across layers. |
| `examples/implicit-contract-break.md` | A refactor appears signature-safe but may change observed caller behavior. |
| `examples/test-manipulation.md` | Tests, snapshots, or type checks are tempting to change just to make verification pass. |
