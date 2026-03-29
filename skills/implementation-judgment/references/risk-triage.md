# Intervention Triage

Use this when the Judgment Dimensions in `SKILL.md` have raised concerns but
the scope decision (narrow patch vs. focused improvement vs. pause) is not yet
clear.

## Narrow Local Patch Is Usually Enough When

Prefer a narrow patch when most of these hold:
- the change is well-bounded and local
- the relevant code has clear ownership and a coherent existing pattern
- the smallest responsible edit does not add duplication, coupling, or confusion
- existing structure already supports the new behavior cleanly
- no Judgment Dimension raised a concern that a patch would deepen
- future related changes are not made meaningfully harder

A narrow patch is not "bad" just because it is small. It is the right choice
when small and structurally honest are the same thing.

## Focused Structural Improvement Is Warranted When

A limited structural improvement is warranted when the Judgment Dimensions
surface one of these specific patterns:

**Abstraction Correctness signals:**
- The same logic appears in 3+ places with identical semantics → extract
- A shared abstraction has accumulated 3+ conditional parameters → consider inlining and re-extracting
- Two duplicates are diverging (different change reasons, different rates of change) → keep separate

**Complexity Placement signals:**
- A concern (validation, error handling, transformation) is scattered across 3+ layers with no single layer owning it completely → consolidate into the natural owning layer
- A generic utility has acquired domain-specific logic → extract the domain logic out, restore the utility's generality
- Callers need to know implementation details to use a function correctly → the interface is too shallow, pull complexity downward

**Contract and Compatibility signals:**
- A behavioral change is disguised as a refactoring → make it explicit (new name, new function, or documented migration)
- Side effects that callers depend on are being removed → preserve them or add explicit migration

**Dependency Direction signals:**
- A stable module is importing a volatile one → reverse the dependency or introduce an interface between them
- Two modules change together but have no explicit dependency → consider merging or introducing a shared abstraction

The improvement should stay tightly coupled to the current change. Do not
broaden the work just because a cleaner architecture is imaginable.

## Pause and State the Gap When

Stop and make the uncertainty explicit when:
- you do not yet understand why the current code is shaped the way it is
- multiple Judgment Dimensions point in conflicting directions (e.g., Abstraction Correctness says extract, but Deletability says keep separate)
- the touched area may be protecting compatibility, migration ordering, or shared-contract behavior that is still unclear
- you are about to "clean up" code without enough evidence that the cleanup is safe
- the implementation choice could reshape ownership boundaries, but the existing ownership model is not yet understood
- you cannot identify the implicit contracts that callers depend on

When this happens, do not guess confidently. State what is unclear, why it
matters, and what understanding is missing.

## Concrete Escalation Heuristics

**Stronger intervention more likely justified:**
- change touches shared contracts or default behavior
- same logic is repeated in 3+ places with identical semantics
- a wrong abstraction has accumulated 3+ conditional parameters
- ownership boundaries between modules are unclear and the change affects them
- error handling is scattered with no clear owner
- the change crosses a package or module boundary

**Stronger intervention less likely justified:**
- change is truly local and single-purpose
- duplication exists in only 2 places and the copies are diverging
- existing structure already supports the new behavior cleanly
- the alternative to patching is a broad redesign that goes beyond what this change needs

## Failure Pattern Summary

The most common failure pattern across all Judgment Dimensions:

> Making the patch pass now while quietly making future changes harder.

Specific forms:
- Adding a conditional to a shared abstraction instead of questioning the abstraction
- Scattering a concern across layers instead of finding the owning layer
- Changing behavior under an old name instead of introducing a new contract
- Adding a dependency from stable code to volatile code
- Removing duplication between cases that are actually diverging

## Rule of Thumb

If a tiny patch would distort responsibilities, it is too small.
If a broad redesign goes beyond what this change needs, it is too large.
If you cannot trace the implicit contracts that callers depend on, you are
not yet ready to change the code.
