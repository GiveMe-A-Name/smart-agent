# Intervention Triage for `engineering-implementation-thinking`

Use this only when the main `SKILL.md` is not enough.

This reference helps decide how much engineering intervention the current change warrants:
- a narrow local implementation
- a focused structural improvement
- an explicit pause because understanding is still too weak

The goal is not to justify bigger changes.
The goal is to avoid both under-reacting and over-reacting.

## Narrow Local Implementation Is Usually Enough When

Prefer a narrow implementation when most of these are true:
- the requested change is well-bounded and local
- the relevant code has clear ownership and a coherent existing pattern
- the smallest responsible edit does not add duplication, coupling, or long-term confusion
- existing structure already supports the new behavior cleanly
- future related changes are not made meaningfully harder by a local patch

A narrow implementation is not "bad" just because it is small.
It is the right choice when small and structurally honest are the same thing.

## Focused Structural Improvement Is Warranted When

A limited structural improvement is warranted when a purely local patch would likely:
- duplicate logic that should stay unified
- push responsibility into the wrong module, layer, or abstraction boundary
- preserve a pattern that is already actively causing confusion or fragility in the touched area
- make the next related change noticeably harder
- hide an important contract, default, or dependency in an even less maintainable way
- solve the current request while making the surrounding code less coherent

The improvement should stay tightly coupled to the current change.
Do not broaden the work just because a cleaner architecture is imaginable.

## Pause And State The Gap When

Stop and make the uncertainty explicit when:
- you do not yet understand why the current code is shaped the way it is
- multiple implementation directions remain plausible for materially different reasons
- the touched area may be protecting compatibility, migration, ordering, or shared-contract behavior that is still unclear
- the agent is about to "clean up" code without enough evidence that the cleanup is safe or appropriate
- the implementation choice could reshape ownership boundaries, but the existing ownership model is not yet understood

When this happens, do not guess confidently.
State what is unclear, why it matters, and what understanding is missing.

## Escalation Hints

A stronger intervention is more likely to be justified when the change touches:
- shared contracts or default behavior
- repeated logic that should remain consistent
- ownership boundaries between modules, packages, or layers
- compatibility-sensitive or migration-sensitive paths
- historically messy areas where another narrow patch would deepen the local debt

A stronger intervention is less likely to be justified when the change is:
- truly local and single-purpose
- already well-supported by the existing structure
- easy to implement cleanly without spreading logic or weakening boundaries

## Failure Patterns

Watch for these failure modes:
- using "minimal change" as an excuse to ignore structural damage
- using "maintainability" as an excuse to redesign beyond the real need
- assuming old code is bad without understanding what constraint it was solving
- preserving old code blindly without checking whether the current change exposes a real design weakness
- making the patch pass now while quietly making future changes harder

## Rule Of Thumb

Choose the least invasive implementation that is still honest about the structure the change really needs.

If a tiny patch would distort responsibilities, it is too small.
If a broad redesign goes beyond what this change needs, it is too large.
