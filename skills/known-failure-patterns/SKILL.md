---
name: known-failure-patterns
description: "Use when a known agent-side failure pattern must be prevented from recurring by choosing or creating a project rule, automated check, or tool."
---

# Known Failure Patterns

Use this skill after an agent-side mistake class is known. Do not preserve the incident as a reminder; create or strengthen a prevention mechanism that would have blocked the same mistake class before it happened.

## Principles

- **Prevent the mistake class, not the incident.** State the recurring wrong agent decision independently from the command, output, or review comment that exposed it [because incident details age out, while the decision pattern is what recurs].
- **Choose the highest faithful enforcement layer.** Prefer the strongest layer that can enforce the real constraint: text rule, automated check, or tool/script [because under-escalation leaves preventable mistakes to agent memory, while over-escalation with a weak proxy creates false confidence].
- **Preserve enforcement fidelity.** A higher layer only counts as stronger when it enforces the constraint as stated, not a mechanically convenient proxy [because a proxy check can pass while the original failure mode remains unguarded].
- **Remove the error-prone surface when the surface is the cause.** If the agent understood the rule but the operation invites repeated manual mistakes, replace the operation with scaffolding, generation, or a single correct command [because another reminder does not change the task shape that caused recurrence].
- **Strengthen existing coverage before adding a new entry.** If a project rule, test, or tool already covers the same mistake class, improve that mechanism instead of adding a parallel one [because duplicate prevention entries dilute the signal and make future agents guess which one governs].
- **Make the mechanism usable without conversation context.** The final rule, check, or tool spec must name the trigger, the required behavior, and the failure condition directly [because future invocations usually happen without the incident discussion].

## Constraints

- Stop if the failure class is not known. If the statement only says "the command failed", "the output was wrong", or "the agent missed something", identify the agent-side wrong decision before choosing a prevention mechanism.
- Do not record one-off environment problems, flaky external dependencies, or operator actions as agent failure patterns.
- Before selecting Level 1, state why no faithful mechanical check can assert the constraint.
- Before accepting Level 2, name the exact failure condition the check asserts and compare it to the original mistake.
- Before accepting Level 3, name the manual step or error-prone surface that no longer exists after the tool is used.
- Do not batch unrelated mistake classes into one broad prevention entry.
- Before adding a new mechanism, check existing project rules, tests, and tools for coverage of the same mistake class.
- Before finishing, verify that the mechanism would have caught the original mistake if it had existed beforehand.

## Enforcement Levels

Choose the highest feasible level that passes enforcement fidelity, then choose the minimum sufficient mechanism inside that level.

### Level 1 — Text Rule

Select Level 1 when the mistake depends on intent, convention, domain context, naming, output convention, or behavioral contract that cannot be faithfully verified by a script.

Use this format:
```md
**Pattern**: <one-line description of the recurring mistake class>
**Trigger**: <observable situation or context where the mistake occurs>
**Rule**: <what to do or not do, stated precisely enough to act on without extra context>
```

Pass condition: an agent can apply the rule immediately after reading it. "Use Polars, not pandas, for DataFrame operations in backtest/" passes. "Use the right library" fails.

### Level 2 — Automated Check

Select Level 2 when the mistake is structural and mechanically verifiable in CI or a local check. Choose the smallest test or static check that asserts the real constraint.

Level 2 fits module boundary violations, missing required CLI flags, disallowed dependencies, non-idempotent writes, missing generated artifacts, or other constraints a machine can assert.

Common mappings:
- Cross-layer imports -> AST-based boundary test such as `test_module_boundaries.py`
- Missing `--json` on automation-facing scripts -> CLI contract test
- Non-idempotent database write -> integration test that runs the write twice and asserts stable row count and updated values
- Missing type annotations -> static analysis rule in the project linter config

### Level 3 — Tool or Script

Select Level 3 when the recurring operation itself exposes an error-prone manual surface: repeated boilerplate, multi-step sequences, schema edits, generated contracts, or fragile command construction.

The tool must remove the manual surface by construction, such as scaffolding the correct files, generating the contract, or providing one command with the correct flags and defaults pre-filled.

## Output Shape

When implementation is in scope, edit the project rule, automated check, or tool directly. When implementation is out of scope, produce a prevention specification with:

- **Failure class**: the recurring wrong agent decision, independent from the incident.
- **Existing coverage**: the project rule, check, or tool already covering this class, or "none found".
- **Selected level**: Level 1, Level 2, or Level 3, with the fidelity reason.
- **Mechanism**: the exact rule/check/tool behavior to add or strengthen.
- **Original-mistake test**: why this mechanism would have blocked the observed mistake.

## Examples

**Pass — Level 2 Beats Level 1**

Observed mistake: agents keep adding scripts that print human-readable output where automation expects JSON.

Pass: add a CLI contract test that runs each automation-facing script with `--json` and asserts parseable JSON on stdout. This catches the recurring class mechanically.

Fail: add a text rule saying "support JSON output for automation-facing scripts." This relies on memory even though the interface is testable.

**Boundary — Level 1 is correct**

Observed mistake: agents repeatedly choose a technically valid library that the project avoids for domain reasons.

Pass: add a Pattern / Trigger / Rule entry naming the project area, the avoided library, the preferred library, and the reason. A CI import ban would be too broad if the library remains valid outside that area.

**Fail — Proxy Enforcement**

Observed mistake: agents create database writes that are not idempotent.

Fail: a static check that only searches for the string `ON CONFLICT`. It can pass code that contains the phrase in a comment or in an unrelated query.

Pass: an integration test that runs the write twice and asserts stable row count and updated values, or an AST/query parser check if the query structure is available.
