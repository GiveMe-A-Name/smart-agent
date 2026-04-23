---
name: code-review
description: "Evaluate code changes for correctness, design, and risk. TRIGGER when: code changes exist and the current task is to judge whether those changes are acceptable or what issues they contain. DO NOT TRIGGER when: no code changes exist yet or the task is to argue about already-given review feedback rather than evaluate the changed code."
---

# Code Review

A review that catches nothing is worse than no review — it creates false confidence. The highest-value findings are not bugs in the implementation — they are problems with the change itself: wrong direction, wrong abstraction, wrong scope.

## Trigger Logic

**Invocation default**: Skipping structured review before merge lets problems compound. The cost of unnecessary rigor is a short overhead. The cost of missing it is bugs in production or findings that block merge after the fact.

Use when:
- code changes exist (local diff, staged changes, or remote PR) and the current task is to evaluate those changes
- the task is to decide whether the change is ready to merge, not ready, or acceptable only with fixes

Do not use when:
- the task is to judge or respond to already-existing review comments rather than evaluate the changed code itself [because when the task has shifted to a comment exchange, the target is the discussion — reviewing the wrong target produces arguments rather than code inspection]
- no diff or changed code exists to evaluate yet [because reviewing code that doesn't exist collapses into speculation or design work, which belongs to different skills]

## Capability Boundary

This skill owns: reading code changes, identifying issues across all layers (justification through implementation), calibrating severity, and delivering a review verdict grounded in code actually inspected.

It does NOT:
- implement fixes or refactor reviewed code
- adjudicate a feedback discussion once the task has shifted from reviewing the code change to reviewing the comment exchange
- claim coverage over code that was not read

## Invariants

**Upper-layer problems outweigh lower-layer problems.** Before recording Layer 2-3 findings, decide whether Layer 0 or Layer 1 contains a merge-blocking issue. If it does, surface that issue before lower-layer findings. [Because Layer 3 polish on a change that shouldn't exist is wasted review effort — and it signals approval of the direction.]

**Read context, not just diff hunks.** For every function, method, query, or module cited in findings or verdict reasoning, open the surrounding implementation. If the change modifies a public interface, signature, schema, or call pattern, inspect at least one caller or consumer. [Because a diff only shows what changed — not what the change means to the surrounding system. A function that looks unchanged in the diff may have had its behavioral contract broken by the change next to it.]

**Only give feedback on code you actually read.** Do not claim coverage over code you did not open. If some changed files were not reviewed, state that review limit explicitly. [Because findings on unread code are guesses — and guesses labeled as findings make every grounded finding in the review less trustworthy.]

**Every issue needs: layer, file:line, what is wrong, why it matters.** Add a fix only when it is concrete enough to state without hand-waving. Vague findings ("improve error handling", "this could be cleaner") are not actionable.

**Severity must reflect actual risk.** Label a finding `Critical` only if it matches the severity table: data loss, security vulnerability, broken functionality, blocking regression, or a Layer 0/1 problem that invalidates the change. [Because severity inflation converts every issue into a blocker — implementers start discounting all findings and the review loses authority.]

**Automated check status must be explicit.** If the repository exposes merge-gating or touched-area checks (for example lint, typecheck, test, build, or validation commands for files or behavior changed in the diff), record whether each such check passed, failed, or was not run. Do not return `Ready to merge` when any recorded check failed. [Because a review that concludes "Ready to merge" while a merge gate is failing creates explicit false permission to land through a broken gate — this is worse than no verdict.]

**Give a clear verdict.** Every review ends with exactly one of: `Ready to merge`, `Not ready`, or `Ready with fixes`. [Because an open-ended summary defers the merge decision to the person with the most conflict of interest — the author.]

## Verification Approach

This skill is artifact-type: the review document is the artifact. Completion is verified by self-checking the produced review against the Completion Criteria checklist. The evidence is the review document itself — not the agent's impression that the review was thorough.

A review that satisfies the checklist structurally is not necessarily complete. The checklist items are proxies. The underlying test: could a developer read this review, know exactly what needs to change, and make a confident merge decision?

## Decision Signals

Apply review depth and lenses from stable change signals, not from habit.

- Start at Layer 0, then Layer 1, then Layer 2, then Layer 3. If Layer 0 or Layer 1 contains a merge-blocking issue, surface it before cataloging lower-layer issues.
- Apply the Security lens if the diff touches auth, permissions, secrets, crypto, user data access, payments, or deletion paths.
- Apply the Performance lens if the diff changes query shape, caching, concurrency, batching, iteration over variable-size data, or request/response hot paths.
- Apply the Migration lens if the diff changes schema, persisted data shape, rollout ordering, compatibility across versions, or backfill behavior.
- Apply the Dependency lens if the diff adds, removes, or bumps packages, SDKs, build tooling, or external service clients.
- Apply the API Design lens if the diff changes public endpoints, RPCs, events, CLI flags, config schema, exported functions, or exported types.
- Apply user-defined review standards only when a concrete finding maps to one of those dimensions; name the dimension when you use it.
- Apply AI-generated-code checks if the diff contains generated markers, large repetitive edits, mechanically repeated patterns, or machine-produced comments/prose.

## Failure Signals

Stop and reassess if:

- you are reviewing implementation before writing a Layer 0 or Layer 1 assessment of what the change is supposed to do
- you cited a finding on a function, method, query, or module without opening surrounding context
- the diff changes a public interface, signature, schema, or call pattern and you did not inspect at least one caller or consumer
- you are about to write "looks good" or approve without having read the code
- every finding is labeled `Critical` but one or more findings do not match the `Critical` severity definition
- all your findings are Layer 3 and you found nothing at Layers 0-2 — either the change is genuinely excellent, or you didn't look hard enough at the higher layers
- findings lack layer, file:line, what is wrong, why it matters, or the lens/standard name when that attribution is what makes the finding defensible
- the review ends without exactly one verdict — an open-ended summary is not a review
- you are giving feedback on code outside the diff — you are responsible for what you reviewed, not what you assumed

## Completion Criteria

- [ ] The review contains an explicit Layer 0-1 assessment, even if the conclusion is that no issue was found at those layers.
- [ ] For every function, method, query, or module cited in findings or verdict reasoning, surrounding context was opened; if a public interface, signature, schema, or call pattern changed, at least one caller or consumer was inspected.
- [ ] Every issue includes layer, file:line, what is wrong, and why it matters; severity labels match the severity table.
- [ ] Any issue whose reasoning depends on a specialized lens or user-defined standard names that lens or standard explicitly.
- [ ] Each specialized lens whose change trigger was present was applied, and any finding that relies on a specialized lens or user-defined standard names it.
- [ ] AI-generated-code checks were applied when generated markers or machine-like edit patterns were present.
- [ ] Merge-gating or touched-area automated checks are listed with pass/fail/not-run status when such checks exist, and the review ends with exactly one verdict: `Ready to merge`, `Not ready`, or `Ready with fixes`.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the review is real.

This check exists because a review that looks complete is not the same as a review that is complete. A findings list can be structurally correct — right format, correct layer labels, verdicts present — while covering code that was never opened. This section externalizes that checkpoint.

Did I mistake superficial diff reading for sufficient context?

Am I exiting because the review is genuinely complete, or because the current findings list looks complete enough?

Completion-faking signals specific to code review — stop if any apply:

- Layer 0-1 assessment was formed from the PR description alone without opening the primary changed files — the description explains intent, not whether the implementation achieves it
- A finding meets the Critical definition (data loss, security, broken functionality, Layer 0/1 invalidation) but was labeled Important or Minor to avoid blocking the change
- Feedback was given on a function's behavior without opening the function — the function name or the diff suggested the behavior, but the implementation was not read
- The verdict is "Ready with fixes" when a Critical finding is present — Critical means must fix before merge, which is "Not ready"
- For AI-generated code: external API calls were noted as "looks reasonable" without spot-checking the actual documentation or source — AI-generated code hallucinates plausible APIs that do not exist

---

## The Layered Review Model

Work from the most consequential question down. **A problem at a higher layer is almost always more important than a problem at a lower layer.**

| Layer | Key question | Critical signals |
|---|---|---|
| **0: Change Justification** | Should this change exist? | Real problem with evidence. Direction aligns with system trajectory. Scope is focused — one thing. |
| **1: Design and Approach** | Is this the right solution? | Simpler alternative exists? Over/under-engineered? Second-order effects (caller impact, migrations, new deps) traced? System health improves or degrades? |
| **2: Implementation Correctness** | Does the code do what it claims? | Caller impact traced for every changed signature. Error handling present. Security and concurrency considered. |
| **3: Maintainability and Tests** | Will this hold up? | Tests exercise real behavior (not mocks) and fail when code breaks. Complexity allows future modification. |

For deep guidance on each layer's key questions and review navigation, see `references/layered-review-depth.md`.

**Depth calibration by risk:**

| Change characteristics | Depth |
|---|---|
| Auth, payments, data deletion, crypto, user-facing security | Maximum — read every line, trace every path |
| New feature, new API surface, schema migration | High — full Layer 0-3 review |
| Bug fix with clear scope and existing test coverage | Medium — verify fix, check for similar bugs, verify test |
| Config change, dependency bump, documentation update | Light — verify intent, check for obvious errors |
| Generated code, formatting-only, trivial renames | Scan — verify the change is what it claims to be |

**Severity calibration:**

| Severity | What qualifies | Default action |
|---|---|---|
| Critical | Data loss, security hole, broken functionality, blocking regression, Layer 0/1 problems that invalidate the change | Must fix before merge |
| Important | Missing error handling, test gaps, unmet requirements, design concerns, unaddressed caller impact | Should fix; defer only with written justification |
| Minor | Style, naming, optimization, documentation | Author decides |

---

## Output Format

See `templates/review-output.md` for the default output structure. Adapt section names or ordering if the same information remains explicit.

**Minimum output:** Layer 0-1 assessment · Issues by severity (or explicit statement that none were found) · Automated check status when merge-gating or touched-area checks exist · Verdict

**For each issue:**
- Layer reference
- `File:line` reference
- Lens or standard name when the finding depends on one
- What is wrong / Why it matters / How to fix (if not obvious)

**Verdict options:** `Ready to merge` / `Not ready` / `Ready with fixes`
