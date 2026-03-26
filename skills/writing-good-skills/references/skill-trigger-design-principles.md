# Skill Trigger Design Principles

## The Core Problem

Skill trigger logic depends on the very judgment the skill is meant to protect.

An AI uses its own intuition to judge "this is a simple problem" → skips the skill → but that judgment itself is the result of shallow thinking. This is a self-validating error loop.

The common pattern that causes this: **"Do not use" conditions depend on properties that can only be confirmed after investigation.**

For example:
> "the change is tiny, local, and has one obvious low-risk path"

The AI uses this condition to decide whether to load the skill — before reading the code. But "whether it's tiny" and "whether there's an obvious low-risk path" are exactly the things you only know after reading the code.

---

## Three Design Directions

The three directions operate at different layers:

| Direction | Layer | Standalone Effect | Combined Value |
|-----------|-------|-------------------|----------------|
| Direction 3 (flip the default) | Decision default | High | Core principle |
| Direction 1 (observable conditions) | Decision content | Medium | Required to land Direction 3 |
| Direction 2 (write skip reasons) | Decision visibility | Low | Debugging tool, helps iterate trigger conditions |

### Direction 1 — Replace "Do not use" conditions with observable facts

Current (requires judgment):
> "the change is tiny, local, and has one obvious low-risk path"

Replace with (verifiable):
> "the entire change is confirmed to fit within a single function, with no calls to or from other modules"

The word **"confirmed"** is key — it requires the AI to have read the code before the condition can be satisfied, rather than skipping based on impression.

**Rule**: "Do not use" conditions must be facts confirmable from already-seen code evidence — not assessments made before investigation.

### Direction 2 — Require AI to state why it is skipping

Add to skill invariants:
> "If you are not loading this skill, state why in your first response."

This forces the meta-cognitive step of articulating the skip reason. However, this only adds transparency — it does not change the underlying judgment. An AI can write a plausible-sounding reason that is still wrong (e.g., "the issue already points to `__webpack_require__.e`, so I can proceed directly" — sounds logical, but relies on prior experience rather than code evidence).

Direction 2's real value: as a **debugging tool** to surface which trigger conditions are still imprecise, then fix them with Direction 1.

### Direction 3 — "Uncertain → invoke" as the default

Current implicit logic: uncertain → do not load.

Flip to: uncertain → load.

The asymmetry in error costs makes this the right default:
- **False positive** (loaded unnecessarily): minor overhead, a few extra reasoning steps
- **False negative** (missed when needed): compromised depth throughout all subsequent work, hard to recover

Put this asymmetry directly in the skill description — do not leave it for the AI to infer:

> "The cost of an unnecessary invocation is minor overhead. The cost of a missed invocation is compromised depth throughout all subsequent work. When in doubt, invoke."

---

## How They Work Together

Direction 3 alone is insufficient — "when in doubt, invoke" without defining what "certain enough to skip" looks like just pushes the problem back. The AI will still misjudge "I'm certain this is simple."

Direction 1 defines the "certain to skip" criteria in terms of observable facts, making Direction 3 the fallback for everything that doesn't meet that bar.

**Combined rule**: Use Direction 1 to narrow the "confirmed safe to skip" conditions. For everything else, Direction 3's principle applies: invoke.

---

## Practical Changes Per Skill

### "Invocation default" paragraph (Direction 3)

Add at the top of every skill's Trigger Logic:

> "**Invocation default**: The cost of an unnecessary invocation is [minor overhead]. The cost of a missed invocation is [specific downstream harm]. [When condition X is not confirmed from code evidence, invoke this skill.]"

The downstream harm description should be specific to the skill — not generic.

### "Do not use" conditions (Direction 1)

For each "Do not use" condition, ask: **can the AI confirm this without reading the code?**

- If yes → condition is fine
- If no → rewrite it to require code evidence first, or delete it

Conditions to always delete:
- "the agent is using X as a default warm-up before every action" — this gives too much permission to skip without requiring any evidence
- Any condition with "obvious", "simple", "trivial", "tiny" that is assessed before investigation

### Redirect conditions

When a skill should not be used because a prerequisite is missing, the condition should say **"build that first, then return here"** — not just "do not use this skill." The "return here" is critical: without it, the AI may skip all skills in the chain rather than deferring and coming back.

Do not hardcode specific skill names in redirect conditions. Express the principle (e.g., "build codebase understanding first") not the mechanism. If the right skill doesn't trigger in the right context, that is a problem with that skill's trigger conditions — not something to paper over with explicit cross-references.

---

## Checklist for Reviewing a Skill's Trigger Logic

Before finalizing any skill's trigger conditions, verify:

- [ ] Does the skill have an "Invocation default" paragraph that states the cost asymmetry?
- [ ] Can every "Do not use" condition be confirmed from already-seen code evidence (not impression)?
- [ ] Are there any conditions with "obvious", "simple", "trivial" assessed before investigation? (remove or rewrite)
- [ ] Is there a "default warm-up" condition that gives blanket permission to skip? (remove)
- [ ] Do redirect conditions say "do X first, then return here" rather than just "do not use"?
- [ ] Are specific skill names hardcoded in redirect conditions? (replace with principle descriptions)
- [ ] Does the frontmatter description reflect Direction 3 — "invoke unless confirmed otherwise" rather than "use when substantial enough"?
