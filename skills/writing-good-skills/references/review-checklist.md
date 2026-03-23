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

Common failures:
- repo-local instructions forced into a skill
- project setup material presented as reusable capability

### 3. Constitution

Check:
- are the main constraints explicit?
- does the skill say what must not happen?
- does it encode failure-preventing judgment?

Common failures:
- only steps, no constraints
- no red lines
- no boundary rules

### 4. Execution Guidance

Check:
- does it tell the agent what to do in practice?
- is the workflow minimal and usable?
- are examples teaching judgment rather than just format?

Common failures:
- only philosophy, no applied guidance
- too many repeated examples
- long theory that does not change action

### 5. Verification

Check:
- are there realistic eval prompts or equivalent checks?
- do they target likely failure modes?
- if the skill is non-trivial, is there evidence it changes behavior?

Common failures:
- no evals
- generic evals with no failure pressure
- verification treated as optional polish

### 6. Packaging And Redundancy

Check:
- is the main `SKILL.md` lean but complete?
- do support folders match file roles? Use `references/lean-skill-patterns.md` as the canonical split guide.
- were essential rules mistakenly moved out?
- does more than one section make the same argument?
- was a new section added where an existing section should have been strengthened instead?

Common failures:
- main skill too long and repetitive
- main skill hollow because key logic moved out of the main file
- support files dumped into the wrong folder instead of following the canonical split guide
- repeated packaging advice scattered across multiple sections
- a local fix appends fresh prose without first integrating overlapping material

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
- prioritized fixes
- rewritten text only for the weakest parts

## Minimal Rule

If a review comments only on wording and style, it is incomplete.
