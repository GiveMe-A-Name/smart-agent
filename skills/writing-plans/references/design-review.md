# Design Review Reference

Read this reference only after the Design Review Gate in `SKILL.md` triggers and the smallest useful representation is unclear. `SKILL.md` owns the gate; this file explains how to expose the resulting decision without a fixed field template.

A review needs three things: the proposed shape, the material alternative or binding constraint, and the evidence-based reason for the choice. Do not repeat the Decision Brief or summarize tasks.

## Choose The Form From The Decision

### Responsibility or dependency choice

Use one compact diagram plus the ownership rule:

````markdown
## Design Review

The request boundary authenticates and translates transport data; the permission policy owns authorization decisions; repositories enforce the approved query scope.

```text
request boundary -> permission policy -> scoped repository query
```

Putting permission branches directly in each endpoint is rejected because the same role rule would drift across callers.
````

Do not add a second topology tree or responsibility list when the paragraph and diagram already answer the decision.

### Branching, fallback, or state transition

Use a matrix when reviewers need to approve outcomes:

| Current state | Event | Result |
| --- | --- | --- |
| Pending | Worker accepts | Running |
| Pending | Worker rejects | Failed with reason |
| Running | Timeout | Failed; no automatic restart |

Use pseudocode instead when ordering is the approval issue:

```text
validate request
if invalid: return without persistence
persist accepted record
enqueue work
if enqueue fails: mark record failed and return controlled error
```

Keep only semantics that require approval, not production-shaped code.

### Public contract change

Use a diff and one compatibility statement:

| Contract | Before | After |
| --- | --- | --- |
| `import --strict` | unsupported | optional; omitted preserves current behavior |

An architecture diagram would add no value when the decision is entirely contractual. For a complete mixed-version API example, see `examples/medium-api-contract-change.md`.

### Migration or rollout choice

Use stages with a rollback boundary:

1. Deploy readers that accept both schema versions.
2. Backfill the new field while old writers remain valid.
3. Switch writers to the new schema.
4. Remove old-field reads only after no old writers remain.

State the last stage from which rollback is safe. Do not duplicate these stages as task summaries; tasks add files, commands, and verification.

## Failure Patterns

### Fixed-field completion

Labels such as “clean architecture,” “clear responsibilities,” or “robust handling” do not expose a decision. Delete the review if no gate trigger exists; otherwise replace labels with the specific diagram, matrix, diff, or rollout stages needed for approval.

### Repeated representation

Do not describe one ownership split in a diagram, topology tree, responsibility list, flow paragraph, boundary list, and counterexample. Keep one primary representation and add only information it cannot carry.

### Invented alternatives

Do not create a second design merely to populate an alternatives field. A material alternative must come from repository evidence, user direction, or an actual boundary choice.

For a complete trust-boundary review, see `examples/large-plugin-system.md`. For an over-triggered or content-free review, see `examples/completion-faking-anti-pattern.md`.
