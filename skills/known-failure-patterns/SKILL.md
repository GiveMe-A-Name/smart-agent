---
name: known-failure-patterns
description: "Use when a known agent-side failure pattern must be prevented from recurring by choosing or creating a project rule, automated check, or tool."
---

# Known Failure Patterns

For a known agent-side mistake, do not record a reminder and stop. Select a prevention mechanism that would have blocked the original mistake before it happened.

**Enforcement Fidelity Law**: A higher enforcement layer is only an upgrade if it enforces *the constraint as stated* — not a mechanically convenient proxy for it. A check that is easier to implement than the real constraint is not Level 2 enforcement; it is a weaker constraint dressed as a stronger one, creating false confidence while the actual failure mode remains unguarded. Every escalation decision must pass this test before it counts as valid.

## Trigger Logic

Invoke when the current task asks how to prevent an observed agent mistake from recurring. The failure must already be identified at the level of "what the agent did wrong" rather than only "something failed" [because prevention mechanisms built before root cause is known usually encode the symptom, not the failure class].

- A failure pattern has been identified and needs a durable constraint.
- The same class of mistake has appeared more than once and needs encoding at the right enforcement layer.
- Recent agent behavior is being reviewed to determine which mistakes become project rules, tests, or tools.
- A proposed prevention mechanism needs to be checked for whether it would have caught the original mistake.

The cost of invoking unnecessarily is a short triage pass. The cost of skipping it is recurring correction work and a harness that never learns from observed failures.

Do not invoke when:
- The root cause is still unknown; first identify the agent-side mistake, then return here.
- The failure is a one-off environment problem, flaky external dependency, or operator action with no recurring agent-side decision to constrain.
- The task is ordinary feature work or bug fixing with no observed agent mistake to prevent.

## Boundary

This skill owns:
- Choosing the highest feasible enforcement layer for an observed agent-side failure: project rule, automated test, or tool.
- Specifying the minimum sufficient prevention mechanism within that layer to stop the mistake class from recurring.
- Judging whether the proposed sedimentation is specific, non-duplicative, and strong enough to have caught the original failure.

This skill does not own:
- Root-cause diagnosis when the mistake class is still unknown.
- Recording one-off imperfections that do not justify a durable harness change.
- Treating every prevention decision as a mandatory implementation task. The output may be an implemented rule/test/tool when the current task asks for it, or a concrete prevention specification when implementation is out of scope.

## The Three Sedimentation Levels

The central judgment has two steps: **choose the highest feasible enforcement layer, then choose the minimum sufficient mechanism within that layer.**

A text rule where a faithful test was possible is wasted leverage [because it requires agent memory for a constraint CI could enforce silently]. A test that enforces only a convenient proxy is also wasted leverage [because it creates confidence while leaving the original failure unguarded].

### Level 1 — Text Rule

Select Level 1 when the mistake depends on intent, convention, or domain context that cannot be faithfully verified by a script.

Level 1 fits failures involving tech stack preference, idiomatic pattern, domain rule, naming or output convention, or behavioral contract.

**Format**:
```
**Pattern**: <one-line description of the mistake class>
**Trigger**: <what situation or context causes this mistake to occur>
**Rule**: <what to do or not do — stated precisely enough to act on without extra context>
```

Pass condition: an agent can apply the rule immediately after reading it. "Use Polars, not pandas, for all DataFrame operations in backtest/" passes. "Use the right library" fails.

### Level 2 — Automated Check

Select Level 2 when the mistake is structural and can be mechanically verified. Apply the Enforcement Fidelity Law before escalating: the check must enforce the constraint as stated, not a proxy. Choose the smallest test or static check that would reliably catch the mistake class.

Level 2 fits failures involving module boundary violation, missing CLI flag, disallowed dependency, non-idempotent write, or another constraint that CI can assert.

Common patterns:
- Cross-layer imports → AST-based boundary test (e.g., `test_module_boundaries.py`)
- Missing `--json` on a new CLI script → CLI contract test
- Write without `ON CONFLICT DO UPDATE` → idempotency integration test
- Missing type annotations → static analysis rule in ruff config

### Level 3 — Tool or script

Select Level 3 when the mistake recurs because the operation itself exposes an error-prone manual surface: repeated boilerplate, multi-step sequences, schema edits, or generated contracts. Choose the smallest tool or script that removes the error-prone surface by construction.

Level 3 fits failures where the recurring operation can be replaced by scaffolding, generation, or a single command with the correct contract pre-filled.

## Decision Signals

1. **Is the failure class known?** If the statement still says only "the command failed," "the output was wrong," or "the agent missed something," this skill is premature. The usable input is "the agent made this recurring wrong decision under these conditions."

2. **Is this structurally verifiable?** If a script could mechanically detect the mistake in CI, consider Level 2 — but apply the Enforcement Fidelity Law first. If the constraint depends on context or intent, that is evidence no faithful mechanical check exists, and Level 1 is the correct layer.

3. **Is the operation's error-prone surface the root cause, not a misunderstanding?** If the agent understood the rule but the task structure invited repeated mistakes, consider Level 3.

4. **Is the proposed mechanism specific enough to act on immediately, without re-reading the surrounding context?** If not, sharpen it or split it into multiple focused entries.

5. **Does an existing prevention mechanism already cover this class of mistake?** If yes, strengthen the existing mechanism rather than adding a new one [because duplicate entries dilute the signal and make future agents guess which one governs].

6. **Would this mechanism have caught the original mistake if it had existed beforehand?** If not, it is not precise enough yet.

## Invariants

- Escalate to the highest level where enforcement passes the Enforcement Fidelity Law. Both failure directions are real: under-escalation wastes leverage; escalation with a proxy check creates false confidence.
- One mistake class maps to one prevention mechanism. Do not batch unrelated mistakes into a vague rule [because a broad rule cannot be checked against the original failure].
- Text rules must be actionable without forward references or additional context the agent won't have at read time.
- Automated checks must assert the real constraint, not a nearby measurable substitute.
- Tools must remove the recurring error-prone operation; a wrapper that still leaves the same manual steps exposed is not Level 3.
- Never add a new entry for something already covered — strengthen the existing mechanism instead.

## Failure Signals

Stop and revise when:

- The proposed fix defaults to a Level 1 text rule even though a script or CI check could deterministically catch the mistake.
- The write-up describes the specific incident instead of the recurring mistake class that needs to be prevented [because incident details age out, while the class recurs].
- The rule is only understandable in the context of the surrounding conversation and would not be actionable when read in isolation later [because future invocation usually happens without that context].
- Multiple entries partially overlap the same mistake class, creating duplicate or diluted guidance [because future agents will not know which entry is authoritative].
- A one-off environment issue, flaky external dependency, or operator error is being recorded as if it were an agent capability failure [because the harness cannot prevent causes it does not control].
- The proposed entry would not actually have caught the original mistake if it had existed beforehand.
- A tool-worthy failure mode is being treated as documentation debt instead of eliminating the error-prone operation itself.
- The escalation to Level 2 fails the Enforcement Fidelity Law: the check enforces a proxy, not the constraint as stated.

## Completion Criteria

- [ ] The agent-side failure class is stated independently from the incident that exposed it.
- [ ] The selected level is the highest feasible layer that passes the Enforcement Fidelity Law.
- [ ] **If Level 1**: the final entry follows Pattern / Trigger / Rule format, and the Rule can be acted on without reading the conversation.
- [ ] **If Level 2**: the check names the exact constraint, where it runs, and the failure condition it asserts; if implementation is in scope, the check has been added.
- [ ] **If Level 3**: the tool names the manual/error-prone surface it removes; if implementation is in scope, that surface is no longer required for the recurring operation.
- [ ] The mechanism strengthens any existing coverage instead of duplicating it.
- [ ] The mechanism would have caught the original mistake if it had existed beforehand.

## Verification Approach

This skill is artifact-type. Verify completion by checking the chosen prevention mechanism or prevention specification against the Completion Criteria. No user confirmation is required unless the task explicitly asks for approval before editing project rules, tests, or tools.

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

## Anti-Rationalization Check

Before exiting, pause on these:

- Does my level choice pass the Enforcement Fidelity Law — in both directions?
- Is the rule I wrote specific enough to act on without extra context — or does it assume knowledge the agent won't have?
- Am I recording a genuine recurring risk, or a one-off that is unlikely to recur?
- Would the entry, as written, have prevented the original mistake? Or does it only gesture at the problem?
