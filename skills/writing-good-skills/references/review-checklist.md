# Review Checklist

## Review Order

When assessing an existing skill, review it in this order:

1. `description`
2. scope boundary
3. constitution
4. execution guidance
5. verification
6. packaging

This order matters. A skill that is well-written but poorly triggered is still weak.

## Section Checks

### 1. Description

Check:
- is it trigger-focused?
- does it describe when to use the skill rather than what steps it performs?
- would it help discovery?

Common failures:
- workflow summary
- vague description like "helps with skills"
- no concrete contexts or symptoms

### 2. Scope Boundary

Check:
- is this really a reusable capability?
- should it be `AGENTS.md`, `README.md`, or a note instead?

Common failures:
- repo-local instructions forced into a skill
- project setup material presented as reusable agent capability

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

### 6. Packaging

Check:
- is the main `SKILL.md` lean but complete?
- does `references/` hold only supporting material?
- were essential rules mistakenly moved out?

Common failures:
- main skill too long and repetitive
- main skill hollow because key logic moved to references
- `references/` used as a dump

## Recommended Review Output

Use this shape by default:
- overall assessment
- what already works
- highest-impact gaps
- section review for description, scope, constitution, execution, verification, packaging
- prioritized fixes
- rewritten text only for the weakest parts

## Minimal Rule

If a review comments only on wording and style, it is incomplete.
