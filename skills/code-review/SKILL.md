---
name: code-review
description: "Evaluate changed code for correctness, design, and merge risk. TRIGGER when a diff, staged changes, local changes, or PR exists and the task is to review, audit, approve, or judge the changed code. DO NOT TRIGGER when no changed code exists or the task is responding to existing review feedback rather than evaluating the change."
---

# Code Review

Review changed code as a merge decision, not a style pass. The highest-value findings are problems with the change itself: wrong direction, wrong abstraction, broken contract, missing caller impact, unsafe rollout, or verification that does not prove the behavior.

## Principles

- **Review the change, not only the diff.** A diff shows what moved; it does not show caller contracts, existing invariants, or whether the change solves the stated problem [because many regressions are created in unchanged code that now receives different inputs, errors, ordering, or side effects].
- **Start with direction before implementation.** A clean implementation of the wrong change is still a merge blocker. Check Layer 0 and Layer 1 before cataloging Layer 2-3 polish [because lower-layer comments can imply approval of a change that should not land].
- **Treat descriptions as intent, not proof.** PR descriptions, commit messages, issue text, and author claims explain what the change is trying to do; code and tests prove whether it does that.
- **Claim only the code you inspected.** Findings and verdicts must be grounded in files, functions, callers, tests, or checks read in this session. If changed files were not reviewed, state the coverage limit.
- **Findings need evidence and consequence.** Every issue must name the layer, file/line, specific defect, and why it matters to correctness, security, data, compatibility, operability, or maintainability.
- **Severity follows merge risk.** Critical means the change must not merge as-is: data loss, security hole, broken existing functionality, blocking regression, or a Layer 0/1 flaw that invalidates the change. Important means should fix unless the owner explicitly accepts the risk. Minor means author discretion.

## Constraints

- Do not perform a code review without changed code to inspect. If no diff, local changes, staged changes, or PR exists, this skill does not apply.
- Before reviewing implementation details, identify the intended change from the PR description, commit messages, issue text, or changed file list. If the intent cannot be stated from available evidence, make that a Layer 0 issue or ask for clarification.
- For every function, method, query, module, test, or API named in a finding or verdict reason, open the surrounding implementation. A finding based only on a hunk, filename, function name, or author description is not grounded.
- If the change modifies a public interface, exported type/function, API endpoint, event, schema, config shape, CLI flag, error contract, side effect, or call pattern, inspect at least one caller, consumer, or compatibility boundary before approving.
- Do not give feedback on unrelated code outside the diff unless the changed code directly depends on it or breaks its contract. If adjacent code is merely messy, omit it or mention it as out of scope.
- Apply specialized lenses only when the diff triggers them. If a lens affects a finding, name the lens; if a high-risk lens is triggered and produces no finding, note what was checked.
- If repository merge gates or touched-area checks exist, report their status as Passed, Failed, or Not run. Do not return `Ready to merge` while a relevant recorded check failed.
- Do not implement fixes while reviewing unless the user explicitly asks for changes rather than review.
- Do not use this skill to decide how to respond to existing review feedback. That task reviews a feedback exchange, not the changed code itself.

## Layered Review

Review from the highest-impact layer downward. If Layer 0 or Layer 1 blocks the change, surface that before spending review budget on Layer 3 polish.

| Layer | Question | Signals |
|---|---|---|
| **0: Change Justification** | Should this change exist? | Problem is real and evidenced; direction matches product/system trajectory; scope is focused; no unrelated changes bundled. |
| **1: Design and Approach** | Is this the right solution? | Simpler alternative exists; abstraction is over/under-built; caller impact, migration, dependency, or rollout effects are traced; system health improves rather than degrades. |
| **2: Implementation Correctness** | Does the code do what it claims? | Logic, edge cases, error handling, concurrency, security, state, and caller compatibility work under realistic inputs and failure modes. |
| **3: Maintainability and Tests** | Will this hold up? | Tests fail when behavior breaks; code remains understandable; complexity, naming, docs, and test structure do not hide future defects. |

Severity calibration:

| Severity | What qualifies | Verdict impact |
|---|---|---|
| **Critical** | Data loss, security vulnerability, broken existing functionality, blocking regression, or Layer 0/1 invalidation | `Not ready` |
| **Important** | Missing error handling, test gap, unmet requirement, design concern, untraced caller impact, risky rollout gap | `Ready with fixes` or `Not ready` depending on blast radius |
| **Minor** | Style, naming, documentation, localized readability, non-blocking cleanup | Author discretion |

## Review Lenses

- **Security:** auth, authorization, permissions, secrets, crypto, user data access, payments, file upload, deletion, user-controlled input, elevated privileges.
- **Performance:** query shape, caching, concurrency, batching, variable-size iteration, request/response hot paths, data pipelines.
- **Migration/Data Safety:** schema, persisted data shape, rollout ordering, compatibility across versions, backfill, rollback, config format.
- **Dependency:** added/removed/bumped packages, SDKs, build tooling, transitive dependency shifts, external service clients.
- **API Design:** public endpoints, RPCs, events, CLI flags, config schema, exported functions, exported types, error formats.
- **AI-generated code:** generated markers, large repetitive edits, mechanically repeated patterns, machine-produced comments/prose, suspiciously plausible external API calls, tests that assert mocks or restate implementation.

See `references/specialized-lenses.md` when a triggered lens needs deeper domain checks.
See `references/ai-generated-code.md` when generated-code signals are present.
See `references/user-review-standards.md` when a finding maps to deep modules, self-explanatory interfaces, abstraction timing, DRY, open/closed design, or complexity sources.
See `references/layered-review-depth.md` when the layer to inspect first is unclear or the change spans multiple layers.

## Output

Use `templates/review-output.md` for the full structure when writing a complete review.

Minimum output:

- **Change Assessment:** Layer 0-1 assessment, even when no issue is found.
- **Issues:** grouped by severity, or an explicit statement that no issues were found.
- **Automated Checks:** pass/fail/not-run status when merge gates or touched-area checks exist.
- **Verdict:** exactly one of `Ready to merge`, `Ready with fixes`, or `Not ready`.

Each issue must include:

- layer
- file/line
- lens or user-defined standard when it is part of the reasoning
- what is wrong
- why it matters
- fix when the issue does not already determine the required change

## Stop And Revise

Stop and revise the review if any condition is true:

- Layer 0-1 assessment was formed only from the PR description and file list without opening the primary changed files.
- A finding names a function, method, query, module, or behavior whose surrounding implementation was not read.
- A changed public interface, schema, API, event, config, CLI flag, or error contract was approved without checking a caller or consumer.
- All findings are Layer 3 or Minor while Layers 0-1 were not checked.
- A finding lacks layer, file/line, what is wrong, or why it matters.
- A triggered high-risk lens is absent from the review with no finding and no note about what was checked.
- A Critical finding is labeled Important or Minor.
- The verdict is `Ready with fixes` while a Critical finding remains.
- The verdict is `Ready to merge` while a relevant recorded automated check failed.
- The review claims coverage over changed files that were not read.
