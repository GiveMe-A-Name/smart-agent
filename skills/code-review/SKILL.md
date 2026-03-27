---
name: code-review
description: Use when the task is to evaluate code changes for correctness, quality, and production readiness — local working-tree diffs or remote PRs. Do not use when review feedback already exists and the task is deciding how to respond to it, or for reviewing non-code artifacts.
---

# Code Review

A review that catches nothing is worse than no review — it creates false confidence. Every finding must be earned: read the code, locate the issue, state why it matters.

## Trigger Logic

**Invocation default**: Skipping structured review before merge lets problems compound. The cost of unnecessary rigor is a short overhead. The cost of missing it is bugs in production or findings that block merge after the fact.

Use when:
- code changes exist (local diff, staged changes, or remote PR) and the task is to evaluate them
- completing a task in subagent-driven development and review is the gate before the next step
- user asks to "review", "check", or "audit" code changes

Do not use when:
- review feedback already exists and the task is deciding what to implement, challenge, or clarify — that is a feedback evaluation task, not a code review
- the task is planning, implementing, or debugging — this skill evaluates finished work, not work in progress

## Capability Boundary

This skill owns: reading code changes, identifying issues, calibrating severity, and delivering a structured verdict.

It does NOT:
- implement fixes or refactor reviewed code
- verify that previously given feedback was applied — that belongs to the next review cycle
- claim coverage over code that was not read

## Invariants

**Understand intent before reading the diff.** Read the PR description, linked spec, plan, or commit messages first. A reviewer who does not know what the change is supposed to accomplish cannot tell whether the implementation is correct — only whether it compiles.

**Read context, not just diff hunks.** For every changed function or module, read enough surrounding code to understand: what callers expect, what invariants the module maintains, and what the changed lines actually do in context. A diff line that looks correct in isolation often reveals a problem when read against the code that calls it.

**Only give feedback on code you actually read.** Never approve or comment on code outside the diff or files explicitly opened. "Looks good" without reading is a false safety signal.

**Every issue needs: file:line, what is wrong, why it matters.** Vague findings ("improve error handling", "this could be cleaner") are not actionable.

**Severity must reflect actual risk.** Critical means data loss, security vulnerability, broken functionality, or blocking regression — not style, not preference. Inflating severity makes all feedback less trustworthy.

**Give a clear verdict.** Every review ends with: Ready / Not ready / Ready with fixes. An ambiguous conclusion defers the merge decision to the author, which is the reviewer's job.

## Decision Signals

**What to examine:**

- *Intent alignment* — Does what was implemented match what was intended? Restate the stated goal, then assess whether the code achieves it. Misalignment between intent and implementation is often more significant than any individual bug.
- *Necessity* — For each structural element in the diff (parameter, abstraction, helper, config option, exported symbol): what usage evidence justifies its presence? Theoretical justification ("supports testability", "could be useful") does not qualify. Evidence is actual usage by production callers. Test files are not production callers. Test coverage is not usage evidence — a test proves that an element exists and behaves as specified, not that it should exist. When a code element has tests but no production callers, the tests are artifacts of its existence, not justification for it — both the element and its tests are candidates for removal.
- *Caller and usage impact* — How is the changed code called? If a function signature, return type, or behavior changed, trace to its callers and check whether they handle the new behavior correctly. Changes that look self-contained often break at the call site.
- *Correctness* — Does the code do what it claims? Are edge cases and error paths handled? Are there logical errors?
- *Security* — Injection vectors, unvalidated inputs, exposed secrets, privilege escalation, unsafe deserialization?
- *Error handling* — Are errors caught at the right layer? Do failures produce useful signals or silent data corruption?
- *Tests* — Do tests cover real logic or only happy paths? Do they test the system itself or just its mocks?
- *Requirements* — Does the implementation match the stated plan or spec? Is anything missing or out of scope?
- *Consistency* — Are there similar patterns elsewhere in the codebase that should have been updated in the same change? A partial fix that leaves the same problem in three other places is not a fix.
- *Architecture* — Does the change fit the existing design? Does it introduce unintended coupling or scalability implications?

**Severity calibration:**

| Severity | What qualifies | Default action |
|---|---|---|
| Critical | Data loss, security hole, broken existing functionality, blocking regression | Must fix before merge |
| Important | Missing error handling, test gaps, unmet requirements, architecture problems | Should fix; defer only with written justification |
| Minor | Style, naming, optimization, documentation improvements | Author decides |

**Remote PR vs local changes:**
- For remote PRs: read the PR description and any existing review comments before starting. Understanding intent prevents misread findings.
- For local changes: `git diff` for unstaged, `git diff --staged` for staged. Clarify scope with the author if it is ambiguous.

**Automated checks:** Run the project's preflight, lint, or test suite when the change is substantial or when test coverage is in question. A passing review with failing automated checks is not a passing review. Skip only when infeasible (no test runner configured, environment not available) — if skipped, say so explicitly in the review.

## Failure Signals

Stop and reassess if:

- you have not read the PR description, plan, or spec before starting — you do not yet know what the change is supposed to do
- you are reviewing diff hunks without having read the surrounding context for each changed function
- you are about to write "looks good" or approve without having read the code
- every finding is labeled Critical — severity inflation makes all feedback untrustworthy
- findings lack file:line references — specificity is the baseline for actionable feedback
- the review ends without a verdict — an open-ended summary is not a review
- you are giving feedback on code outside the diff — you are responsible for what you reviewed, not what you assumed
- a finding describes a vague risk without naming what breaks and why ("this could cause issues")
- you assessed correctness without checking the callers of changed functions — the call site is often where the real problem surfaces

## Output Format

See `templates/review-output.md` for the full output structure.

**Required sections:** Strengths · Issues (by severity) · Assessment with verdict

**For each issue:**
- `File:line` reference
- What is wrong
- Why it matters
- How to fix (if not obvious)

**Verdict options:** `Ready to merge` / `Not ready` / `Ready with fixes`
