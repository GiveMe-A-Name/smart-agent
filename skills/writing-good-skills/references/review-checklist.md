# Review Checklist

## Review Order

Before reviewing a specific section, read the whole main `SKILL.md` once and mark where the same idea already appears.

Review in this order:
1. `description`
2. capability boundary
3. invariants
4. decision signals and applied guidance
5. Self-Check design
6. packaging and redundancy

This order matters. A polished file that triggers badly or repeats itself is still weak.

If the user points at one weak section, do not stay local by default. First check whether the draft already has a better home for the same idea.

## Section Checks

### 1. Description And Trigger Condition Quality

Check the `description` field:
- is it trigger-focused?
- does it describe when to use the skill rather than the workflow?
- would it help discovery?
- is the problem substantial enough that this skill should apply at all, rather than a local wording or formatting fix?
- does it follow the recommended format: "Invoke [when/unless] [observable condition]. Cost of unnecessary invocation: [specific minor cost]. Cost of missing: [specific major harm]."?
- if using "unless", is the exclusion condition trivially confirmable from independent facts (not impression-based)?
- does it prefer "when" over "unless" to avoid rationalization pressure at the first gate?

Common failures:
- workflow summary
- vague wording like "helps with skills"
- no concrete contexts or symptoms
- missing cost asymmetry statement
- "unless" conditions based on impressions ("already obvious", "already confirmed") rather than verifiable facts

Check the Trigger Logic section (see `references/skill-trigger-design-principles.md` for the full checklist):
- does the skill have an **invocation default** paragraph that states the cost asymmetry — what happens when the skill is missed vs. invoked unnecessarily?
- can every **"Do not use"** condition be confirmed from already-seen evidence, not impression? (remove conditions with "obvious", "simple", "trivial", "tiny" assessed before investigation)
- is there a **"default warm-up" or "every action" condition** that gives blanket permission to skip? (remove it)
- do redirect conditions say **"do X first, then return here"** rather than just "do not use"?
- are **specific skill names hardcoded** in redirect conditions? (replace with principle descriptions)
- does the **description** reflect direction-3 — "invoke unless confirmed otherwise" rather than "use when substantial enough"?

### 2. Capability Boundary

Check:
- is this really a reusable capability?
- should it be `AGENTS.md`, `README.md`, or a note instead?
- does it define what the skill owns without turning the boundary into skill-to-skill routing?

Common failures:
- repo-local instructions forced into a skill
- project setup material presented as reusable capability
- skill-to-skill routing used to compensate for a weak or missing boundary

### 3. Invariants

Check:
- are the main constraints explicit?
- does the skill say what must not happen?
- does it encode failure-preventing judgment?
- does it say what the skill does not own when it reaches the edge of its scope?

Common failures:
- only steps, no constraints
- no red lines
- no boundary rules

### 4. Decision Signals And Applied Guidance

Check:
- does it teach the agent what to notice in practice?
- does any action guidance sharpen judgment instead of enforcing a rigid SOP?
- are examples teaching judgment rather than just format?
- does it avoid telling the agent which skill must run next?
- were key judgment calls assessed for whether rules alone are sufficient, or whether contrast examples are needed?
- if an `examples/` folder exists, do the examples show a realistic bad case alongside a corrected version, annotated to name what makes the bad case wrong?

Common failures:
- only philosophy, no practical decision cues
- too many repeated examples
- long theory that does not change action
- action guidance that is really a fixed template or mandatory output shape
- skill-to-skill routing disguised as workflow guidance
- examples folder added without assessing whether the judgment gap actually requires it
- examples that restate rules already clear in the prose, rather than revealing a subtle distinction

### 5. Self-Check Design

Check:
- does the skill have a Self-Check section at the end?
- is it declared in the Invariants section ("Before exiting this skill, you MUST complete the Self-Check section at the end")?
- does it contain 3-5 core items (not 7-9)?
- does it follow the recommended structure:
  - invariants check ("Did I follow all invariants?")
  - failure signals check ("Did I catch any failure/self-correction signals?")
  - 1-2 skill-specific critical checks
  - rationalization check ("Am I exiting because work is genuinely complete, or rationalizing?")
- are the checks phrased as questions that can be answered yes/no or with brief reflection?

Common failures:
- no Self-Check section at all
- Self-Check not declared in Invariants
- too many items (7-9+) creating overhead without improving exit quality
- checks that are too vague or philosophical
- missing the rationalization check
- checks that duplicate what's already in invariants without adding exit-specific judgment

### 6. Packaging And Redundancy

Check:
- is the main `SKILL.md` lean but complete?
- do support folders match file roles? Use `references/lean-skill-patterns.md` as the canonical split guide.
- were essential rules mistakenly moved out?
- does more than one section make the same argument?
- was a new section added where an existing section should have been strengthened instead?
- would the skill still teach a clear capability if named skill references were removed?

Common failures:
- main skill too long and repetitive
- main skill hollow because key logic moved out of the main file
- support files dumped into the wrong folder instead of following the canonical split guide
- repeated packaging advice scattered across multiple sections
- a local fix appends fresh prose without first integrating overlapping material
- the skill only works as part of skill-to-skill routing
- the skill only works when followed as a fixed sequence

## Mandatory Boundary-Vs-Routing Audit

Run this even if the user never mentioned skill relationships.

Check for:
- routing language being used in place of a clear boundary
- named skill references doing skill-to-skill routing work instead of adding real workflow clarity or preventing a severe category error
- scope edges that fail to state what grounding is missing

When found:
- make the boundary explicit first
- keep routing language only when it teaches real workflow knowledge without replacing the boundary
- state what the skill owns and does not own
- keep the next action for the agent to decide unless a named reference is truly necessary to prevent a severe category error

## Mandatory Redundancy Audit

Run this even if the user never said the skill is too long.

Check for:
- the same rule explained in overview, guidance, and checklist
- examples that merely restate bullets
- sections that exist only to justify an already-clear rule

When found:
- keep the clearest version inline
- merge into the strongest existing section before creating a new one
- compress or delete the repeats
- move only the expanded support material to the correct support folder

## Possible Review Deliverables

Choose the smallest deliverable that improves the review outcome:
- overall assessment with highest-impact gaps
- section-by-section review when structure is the problem
- rewritten text only for the weakest parts
- a capability-model rewrite recommendation when the whole file teaches the wrong thing
- a packaging recommendation when structure is the real issue

## Minimal Rule

If a review comments only on wording and style, ignores workflow-routing smells, or treats rigid output templates as harmless, it is incomplete.
