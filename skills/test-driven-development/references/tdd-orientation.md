# TDD Orientation Reference

**Load this reference for**: choosing outside-in vs inside-out before writing the first test.

---

## Before Choosing Orientation

Answer this question first: what design questions in this task can only be resolved by writing the test before the implementation?

- If there are open questions (API shape, where a responsibility lives, how dependencies connect) — the first test must ask those questions. Your choice of outside-in vs. inside-out follows from which question is hardest.
- If there are no open questions — the design is fully pre-decided — state this explicitly. The ordering invariant (tests before implementation) and the caller-perspective invariant still apply; batch TDD is acceptable when all behaviors are enumerable upfront. See `tdd-cycle.md` One Cycle at a Time for when batch is and isn't appropriate.

A common failure: treating "I know what to build" as permission to write tests from the implementation's perspective. The caller-perspective invariant still holds regardless — every assertion must describe what a caller observes, not how the implementation works internally.

---

## Outside-In (London School)

Start at the outermost interface the caller will use. Test the public behavior, use test doubles for collaborators that don't yet exist. Work inward layer by layer.

*Use when:* interface or API design is the hard part; the feature spans multiple layers; you need to discover the right seam before building the internals.

*Trade-off:* Reduces over-engineering risk — you only build what the outer interface requires. Risk is mock overuse if collaborators are doubled too aggressively.

## Inside-Out (Chicago School)

Start from the core domain object or algorithm. Get the inner logic working in isolation first, then integrate outward.

*Use when:* the algorithm or data transformation is the hard part; collaborators already exist and are well-tested; you are building a well-defined calculation or rule engine.

*Trade-off:* Reduces mock-coupling risk — real collaborators are used. Risk is building inner logic that doesn't fit the outer interface without discovering the mismatch until later.

## In Practice

Most real features use both: outside-in to find the right interface, inside-out for complex inner logic. The choice is not a school allegiance — it is a judgment about where the hard problem lives.
