---
name: reviewing-plans
description: "Evaluate an existing implementation plan for intent fit, repository alignment, executability, and hidden risk. TRIGGER when: a plan document exists and the task is to review, audit, approve, or judge it. DO NOT TRIGGER when: creating a new plan, implementing an approved plan, or responding to existing plan-review feedback."
---

# Reviewing Plans

Review a plan as a decision artifact before implementation. Prefer a short, evidence-backed review over a rewrite: identify whether the plan should proceed, what would break if it proceeds unchanged, and the smallest correction needed.

## Principles

- **Intent anchors scope.** Compare the plan's tasks to the stated user-visible goal. Treat missing intent, mismatched intent, or silent scope expansion as design findings [because execution can be technically clean while solving the wrong problem].
- **Repository evidence outranks plan claims.** Before saying a plan fits repo constraints, read the relevant instruction, architecture, convention, or cited code file. If you have not read it, mark the plan assertion as unverified.
- **Review decisions, not implementation style.** Findings should target approach, sequencing, assumptions, interfaces, migration paths, or verification strategy. Omit code-style comments unless the plan depends on that exact code shape.
- **Every finding needs consequence.** Name the plan location, the wrong assumption or decision, and the concrete failure mode. A rule citation alone is not a consequence.
- **Verdict is required.** End with `Approved`, `Approved with fixes`, or `Blocked`. Any Critical issue means `Blocked`.

## Checks

Use these checks in order and stop adding polish once a higher-level issue blocks the plan:

1. **Repo alignment:** Does the plan violate documented constraints, architecture, package boundaries, test/build conventions, or established patterns?
2. **Intent and design:** Does the approach solve the stated goal without hidden scope, unapproved behavior, late cliff-edge risk, or false technical assumptions?
3. **Consistency:** Do tasks fit together in the right order, with compatible interfaces, dependencies, and acceptance criteria?
4. **Executability:** Can a fresh agent start from the plan alone, using exact files/modules, concrete first actions, executable commands, and expected results?
5. **Risk coverage:** Are important edge cases, fallback points, exclusions, and revision triggers named where the plan's approach requires them?

## Constraints

- Do not review a plan you have not read.
- Do not rewrite the plan unless the user explicitly asks; give targeted fixes instead.
- Do not treat plan text as evidence for repository behavior. Read the cited source or label the assertion unverified.
- Cite repository constraint findings with file and line from this session.
- Verification such as "run tests" is non-executable unless it names a command and expected result.
- Use `templates/review-output.md` only when the user asks for a complete formal review; otherwise keep the output compact.

## Output

For each issue, include:

- severity: Critical / Important / Minor
- layer: Repo Alignment / Intent & Design / Consistency / Executability / Risk
- plan location
- what is wrong
- why it matters
- targeted fix

Severity calibration:

- **Critical:** wrong scope, documented constraint violation, wrong files, unverifiable completion, or unapproved user-visible behavior.
- **Important:** likely executable but carries avoidable rework, hidden coupling, ambiguous trade-off, or missing validation of a material assumption.
- **Minor:** wording or detail that improves clarity without changing execution safety.
