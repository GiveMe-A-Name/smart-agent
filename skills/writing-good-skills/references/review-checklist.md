# Review Checklist

## Review Order

Before reviewing a specific section, read the whole main `SKILL.md` once and mark where the same idea already appears.

Review in this order:
1. `description`
2. scope boundary
3. constitution
4. execution guidance
5. verification
6. packaging and redundancy

This order matters. A polished file that triggers badly or repeats itself is still weak.

If the user points at one weak section, do not stay local by default. First check whether the draft already has a better home for the same idea.

## Section Checks

### 1. Description

Check:
- is it trigger-focused?
- does it describe when to use the skill rather than the workflow?
- would it help discovery?

Common failures:
- workflow summary
- vague wording like "helps with skills"
- no concrete contexts or symptoms

### 2. Scope Boundary

Check:
- is this really a reusable capability?
- should it be `AGENTS.md`, `README.md`, or a note instead?
- does it define what the skill owns without turning the boundary into skill-to-skill routing?

Common failures:
- repo-local instructions forced into a skill
- project setup material presented as reusable capability
- skill-to-skill routing used to compensate for a weak or missing boundary

### 3. Constitution

Check:
- are the main constraints explicit?
- does the skill say what must not happen?
- does it encode failure-preventing judgment?
- does it say what the skill does not own when it reaches the edge of its scope?

Common failures:
- only steps, no constraints
- no red lines
- no boundary rules

### 4. Execution Guidance

Check:
- does it tell the agent what to do in practice?
- is the workflow minimal and usable?
- are examples teaching judgment rather than just format?
- does it avoid telling the agent which skill must run next?

Common failures:
- only philosophy, no applied guidance
- too many repeated examples
- long theory that does not change action
- skill-to-skill routing disguised as workflow guidance

### 5. Verification

Check:
- are there realistic eval prompts or equivalent checks?
- do they target likely failure modes?
- if the skill is non-trivial, is there evidence it changes behavior?
- do the evals reward boundary recognition and explicit missing grounding rather than skill-to-skill routing?

Common failures:
- no evals
- generic evals with no failure pressure
- verification treated as optional polish
- evals reward explicit handoff language or named-skill routing

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

## Mandatory Boundary-Vs-Routing Audit

Run this even if the user never mentioned skill relationships.

Check for:
- boundary statements expressed as `hand off to`, `route to`, `use X next`, or similar routing language
- named skill references doing skill-to-skill routing work instead of preventing a severe category error
- scope edges that fail to state what grounding is missing

When found:
- replace routing language with boundary language
- state what the skill owns and does not own
- keep the next action for the agent to decide unless a named reference is truly necessary to prevent a severe category error

## Mandatory Redundancy Audit

Run this even if the user never said the skill is too long.

Check for:
- the same rule explained in overview, workflow, and checklist
- examples that merely restate bullets
- sections that exist only to justify an already-clear rule

When found:
- keep the clearest version inline
- merge into the strongest existing section before creating a new one
- compress or delete the repeats
- move only the expanded support material to the correct support folder

## Recommended Review Output

Use this shape by default:
- overall assessment
- what already works
- highest-impact gaps
- section review for description, scope, constitution, execution, verification, packaging and redundancy
- boundary-vs-routing issues if present
- prioritized fixes
- rewritten text only for the weakest parts

## Minimal Rule

If a review comments only on wording and style, or ignores workflow-routing smells, it is incomplete.
