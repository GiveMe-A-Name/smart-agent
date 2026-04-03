---
name: known-failure-patterns
description: "Invoke when the task is to convert an observed agent mistake or recurring failure mode into a durable prevention mechanism. This skill decides the highest-feasible sedimentation layer for the constraint: a project rule, an automated test, or a tool that removes the error-prone operation entirely. Use when a failure pattern has been identified and the question is how to keep it from recurring."
---

# Known Failure Patterns

Core principle from Harness Engineering:

> *Anytime you find an agent makes a mistake, you take the time to engineer a solution such that the agent never makes that mistake again.*

The key word is **engineer** — not remind, not annotate, not re-explain. The goal is to make the mistake structurally impossible or mechanically detectable, not to document that it happened.

**Enforcement Fidelity Law**: A higher enforcement layer is only an upgrade if it enforces *the constraint as stated* — not a mechanically convenient proxy for it. A check that is easier to implement than the real constraint is not Level 2 enforcement; it is a weaker constraint dressed as a stronger one, creating false confidence while the actual failure mode remains unguarded. Every escalation decision must pass this test before it counts as valid.

## Trigger Logic

Invoke when the current task is to decide how an observed agent mistake should be prevented from recurring.

- A failure pattern has been identified and needs a durable constraint.
- The same class of mistake has appeared more than once and should be sedimented at the right enforcement layer.
- Recent agent behavior is being reviewed to determine which recurring mistakes should become project rules, tests, or tools.

The cost of invoking unnecessarily is a short triage pass. The cost of skipping it is recurring correction work and a harness that never learns from observed failures.

Do not invoke for one-off environment failures or external API errors that have no agent-side root cause.

## Boundary

This skill owns:
- Choosing the highest feasible enforcement layer for an observed failure: project rule, automated test, or tool.
- Specifying the minimum sufficient prevention mechanism within that layer to stop the mistake class from recurring.
- Judging whether the proposed sedimentation is specific, non-duplicative, and strong enough to have caught the original failure.

This skill does not own:
- Implementing the downstream test, tool, or workflow change once the correct sedimentation level has been chosen.
- Diagnosing failures that have no agent-side root cause.
- Recording one-off imperfections that do not justify a durable harness change.

## The Three Sedimentation Levels

The central judgment has two steps: **choose the highest feasible enforcement layer, then choose the minimum sufficient mechanism within that layer.**

A text rule where a test was possible is wasted leverage — it requires the agent to remember, whereas a test enforces silently on every run.

### Level 1 — Text rule (`Known failure patterns` in CLAUDE.md)

**Use when**: the mistake is about intent, convention, or domain understanding — something an agent needs to *know*, but that cannot easily be verified by running a script.

Good for: tech stack preferences, idiomatic patterns, domain rules, naming or output format conventions, behavioral contracts.

**Format**:
```
**Pattern**: <one-line description of the mistake class>
**Trigger**: <what situation or context causes this mistake to occur>
**Rule**: <what to do or not do — stated precisely enough to act on without extra context>
```

A good rule is specific enough that an agent could act on it immediately after reading it. "Use Polars, not pandas, for all DataFrame operations in backtest/" is actionable. "Use the right library" is not.

### Level 2 — Automated test

**Use when**: the mistake is structural and can be mechanically verified. Apply the Enforcement Fidelity Law before escalating: the check must enforce the constraint as stated, not a proxy. Choose the smallest test or static check that would reliably catch the mistake class.

Good for: module boundary violations, missing CLI flags on new scripts, disallowed dependencies, non-idempotent writes.

Common patterns:
- Cross-layer imports → AST-based boundary test (e.g., `test_module_boundaries.py`)
- Missing `--json` on a new CLI script → CLI contract test
- Write without `ON CONFLICT DO UPDATE` → idempotency integration test
- Missing type annotations → static analysis rule in ruff config

### Level 3 — Tool or script

**Use when**: a recurring mistake happens because an operation is *inherently error-prone* for agents — boilerplate that gets wrong in the same ways, multi-step sequences that should be a single command, schema operations that should not be hand-written. Choose the smallest tool or script that removes the error-prone surface by construction.

Good for: scaffolding new modules, initializing DB schema, generating strategy stubs with the correct contract pre-filled.

## Decision Signals

1. **Is this structurally verifiable?** If a script could mechanically detect the mistake in CI, consider Level 2 — but apply the Enforcement Fidelity Law first. If the problem framing indicates the constraint depends on context or intent, that is strong evidence no faithful mechanical check exists, and Level 1 is the correct layer.

2. **Is the operation's error-prone nature the root cause, not a misunderstanding?** If the agent understood correctly but the task structure invited the mistake, consider Level 3.

3. **Is the rule specific enough to act on immediately, without re-reading the surrounding context?** If not, sharpen it or split it into multiple focused entries.

4. **Does a rule already cover this class of mistake?** If yes, strengthen the existing entry rather than adding a new one — duplicate entries dilute the signal.

5. **Would this entry have caught the original mistake if it had existed beforehand?** If not, the entry is not precise enough yet.

## Invariants

- Escalate to the highest level where enforcement passes the Enforcement Fidelity Law. Both failure directions are real: under-escalation wastes leverage; escalation with a proxy check creates false confidence.
- One mistake class → one entry. Do not batch unrelated mistakes into a vague rule.
- Text rules must be actionable without forward references or additional context the agent won't have at read time.
- Never write an entry for something already covered — adds noise, not signal.

## Failure Signals

Stop and revise when:

- The proposed fix defaults to a Level 1 text rule even though a script or CI check could deterministically catch the mistake.
- The write-up describes the specific incident instead of the recurring mistake class that needs to be prevented.
- The rule is only understandable in the context of the surrounding conversation and would not be actionable when read in isolation later.
- Multiple entries partially overlap the same mistake class, creating duplicate or diluted guidance.
- A one-off environment issue, flaky external dependency, or operator error is being recorded as if it were an agent capability failure.
- The proposed entry would not actually have caught the original mistake if it had existed beforehand.
- A tool-worthy failure mode is being treated as documentation debt instead of eliminating the error-prone operation itself.
- The escalation to Level 2 fails the Enforcement Fidelity Law: the check enforces a proxy, not the constraint as stated.

## Completion Criteria

- [ ] The mistake has been assigned to the correct sedimentation level (highest feasible)
- [ ] **If Level 1**: entry follows Pattern / Trigger / Rule format; Rule is specific enough to act on alone
- [ ] **If Level 2**: a test exists or is explicitly planned that would mechanically catch this mistake class
- [ ] **If Level 3**: a tool or script exists or is planned that eliminates the error-prone surface
- [ ] New entry does not duplicate an existing one
- [ ] Entry would have caught the original mistake if it had existed beforehand

## Anti-Rationalization Check

Before exiting, pause on these:

- Does my level choice pass the Enforcement Fidelity Law — in both directions?
- Is the rule I wrote specific enough to act on without extra context — or does it assume knowledge the agent won't have?
- Am I recording a genuine recurring risk, or a one-off that is unlikely to recur?
- Would the entry, as written, have prevented the original mistake? Or does it only gesture at the problem?
