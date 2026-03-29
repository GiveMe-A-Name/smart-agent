---
name: code-review
description: Invoke when the task is to evaluate code changes for correctness, quality, and production readiness — local working-tree diffs or remote PRs. Do not use when review feedback already exists and the task is deciding how to respond to it, or for reviewing non-code artifacts.
---

# Code Review

A review that catches nothing is worse than no review — it creates false confidence. But a review that only catches typos while missing a flawed design is barely better. The highest-value findings are not bugs in the implementation — they are problems with the change itself: wrong direction, wrong abstraction, wrong scope.

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

This skill owns: reading code changes, identifying issues across all layers (justification through implementation), calibrating severity, and delivering a structured verdict.

It does NOT:
- implement fixes or refactor reviewed code
- verify that previously given feedback was applied — that belongs to the next review cycle
- claim coverage over code that was not read

## Invariants

**Upper-layer problems outweigh lower-layer problems.** A change that goes in the wrong direction does not become acceptable because the implementation is clean. A design that introduces unnecessary coupling is not redeemed by thorough tests. Always work from the top layer down — if a higher-layer problem is severe enough, stop and surface it immediately rather than cataloging line-level issues in code that may need to be rewritten.

**Read context, not just diff hunks.** For every changed function or module, read enough surrounding code to understand: what callers expect, what invariants the module maintains, and what the changed lines actually do in context. A diff line that looks correct in isolation often reveals a problem when read against the code that calls it.

**Only give feedback on code you actually read.** Never approve or comment on code outside the diff or files explicitly opened. "Looks good" without reading is a false safety signal.

**Every issue needs: file:line, what is wrong, why it matters.** Vague findings ("improve error handling", "this could be cleaner") are not actionable.

**Severity must reflect actual risk.** Critical means data loss, security vulnerability, broken functionality, or blocking regression — not style, not preference. Inflating severity makes all feedback less trustworthy.

**Give a clear verdict.** Every review ends with: Ready / Not ready / Ready with fixes. An ambiguous conclusion defers the merge decision to the author, which is the reviewer's job.

**Before exiting this skill, you MUST complete the Self-Check section at the end.**

---

## The Layered Review Model

Expert reviewers think in layers, from the most consequential questions down to the least. This is the core of the review — not a checklist to scan, but a sequence of increasingly focused lenses applied to the change.

**The rule: a problem at a higher layer is almost always more important than a problem at a lower layer.** Wrong direction with perfect code is still wrong. Right direction with a few bugs is fixable.

### Layer 0: Change Justification — "Should this change exist?"

This is the question junior reviewers rarely ask and senior reviewers ask first. Before reading a single line of implementation, establish:

**Read the PR description, linked spec, commit messages, or plan.** A reviewer who does not know what the change is supposed to accomplish cannot tell whether the implementation is correct — only whether it compiles.

Then ask:

- **What problem does this change solve?** Can you state it in one sentence? If not, the change may lack focus or the description is insufficient. Ask the author for clarification before proceeding.
- **Does this problem actually need solving right now?** Is there evidence (user reports, metrics, production incidents, product requirements) that this is worth the complexity it introduces? "Might be useful someday" is not justification.
- **Is the direction consistent with where the system is headed?** If the team is migrating away from FooWidget, a change that deepens investment in FooWidget is moving backward — no matter how well-written the code is. Check if the change aligns with known migrations, deprecations, or architectural decisions.
- **Is the scope right?** Does the change do exactly one thing, or does it bundle unrelated modifications that should be separate? A change that mixes a refactor, a feature, and a config tweak is harder to review, harder to revert, and harder to understand in git history.

**If the change should not exist**, stop here. Surface the concern immediately with a constructive alternative ("This area is being deprecated — consider building on BarWidget instead"). Reviewing implementation details of a change that shouldn't land wastes everyone's time.

### Layer 1: Design and Approach — "Is this the right solution?"

The change addresses a real problem. Now: is this the right way to solve it?

- **Is there a simpler approach?** The best code is code that doesn't need to exist. Could this be solved with a configuration change, an existing library, or by removing code rather than adding it? If a 200-line change can be replaced by 20 lines using an existing abstraction, that matters more than whether the 200 lines are well-written.
- **Does the overall structure make sense?** Do the interactions between the pieces of code in the change make sense? Does the new code integrate well with the existing system, or does it sit awkwardly alongside it?
- **Is it over-engineered?** Are there abstractions, parameters, extension points, or generic patterns that have no current callers? "Supports future testability" or "could be useful later" are not justification — the future problem should be solved when it arrives and its actual shape is visible. Flag code that solves speculative problems.
- **Is it under-engineered?** Conversely, does the solution take shortcuts that will force a rewrite soon? A hardcoded list that should be a configuration, a synchronous call that will block under load, a missing abstraction that forces copy-paste in the next feature.
- **What are the second-order effects?** Changes rarely exist in isolation. A new function signature changes the contract for all callers. A new database column affects migration scripts, ORMs, and downstream consumers. A new dependency adds to build time and supply-chain risk. Trace the blast radius.
- **Does this change improve or degrade the system's overall health?** Most systems become complex through many small changes that each seem reasonable in isolation. A change that adds complexity, reduces test coverage, or introduces inconsistency is degrading system health — even if it "works." Don't accept changes that make the codebase worse, even slightly. Hold the line.

### Layer 2: Implementation Correctness — "Does the code do what it claims?"

The design is sound. Now verify the implementation.

- **Correctness.** Does the code actually achieve its stated purpose? Trace the logic path for normal cases, then for edge cases. Look for off-by-one errors, null/undefined handling, incorrect boolean logic, and missing return statements.
- **Caller and usage impact.** If a function signature, return type, error behavior, or side effect changed, trace to every caller. Changes that look self-contained often break at the call site. This is one of the most commonly missed categories in reviews.
- **Error handling.** Are errors caught at the right layer? Do failures produce useful signals (logs, metrics, user-facing messages) or silent data corruption? Is there a catch block that swallows exceptions and returns a default that hides the problem?
- **Security.** Injection vectors (SQL, XSS, command injection), unvalidated inputs, exposed secrets, privilege escalation, unsafe deserialization, TOCTOU race conditions. If the change touches authentication, authorization, or user input, increase scrutiny.
- **Concurrency.** If the code involves shared state, threads, async operations, or distributed coordination: are there race conditions, deadlocks, or ordering assumptions? These are nearly impossible to catch by running the code — they require careful reading.
- **Consistency with codebase patterns.** Are there similar patterns elsewhere in the codebase that should have been updated in the same change? A partial fix that leaves the same bug in three other places is not a fix. Does the change follow existing conventions for error handling, logging, naming?

### Layer 3: Maintainability and Tests — "Will this hold up over time?"

The code works today. Will it be understandable and modifiable six months from now by someone who didn't write it?

- **Complexity.** Can you understand what each function does without re-reading it three times? "Too complex" means "developers are likely to introduce bugs when they try to modify this code." Long functions, deep nesting, clever tricks, and implicit state all contribute.
- **Naming.** Do names communicate what things are and what they do? A good name eliminates the need for a comment. A bad name actively misleads.
- **Comments.** Useful comments explain *why*, not *what*. If a comment explains what the code does, the code should be rewritten to be self-explanatory. But comments about non-obvious business rules, workarounds for external system bugs, or "why not the obvious approach" are valuable.
- **Test quality.** Tests exist — but are they worth having?
  - Do they test real behavior or just exercise mocks?
  - Will they actually fail when the code breaks, or are they so loosely asserted that they pass regardless?
  - Do they cover the important edge cases and error paths, or only the happy path?
  - Are tests clear enough that a failing test tells you what went wrong without debugging?
  - Tests are also code that has to be maintained — don't accept unnecessary complexity in tests.
- **Documentation.** If the change affects how users build, test, or interact with the system, does it update the relevant documentation? If code is deleted or deprecated, is that reflected in docs?

---

## Review Procedure

Don't just read every diff hunk top-to-bottom. Navigate the change deliberately.

### Step 1: Understand the change (Layer 0)

Read the PR description, linked issues, commit messages. Restate the goal in your own words. If you can't, the description is insufficient — ask for clarification before reviewing code.

Scan the file list and overall shape of the change. Is it focused? Is the size reasonable? A change touching 30 files across 5 subsystems is a red flag for scope creep.

**If the change shouldn't exist at all, stop and say so.** Don't review implementation details of a misguided change.

### Step 2: Evaluate the core design (Layer 1)

Find the files that represent the "main" part of the change — usually the file with the most logical changes, not the most lines. Read the core design first. This gives you context for all the supporting changes.

**If there are major design problems, surface them immediately** even if you haven't finished reviewing the rest. The author may be building more work on top of this design. The sooner design issues are raised, the less work is wasted.

### Step 3: Review the implementation (Layers 2-3)

Once the design is confirmed sound, review the remaining files in a logical order. For each changed function, read enough context to understand what the code does in the broader system.

### Calibrating review depth

Not all changes deserve the same scrutiny. Adjust depth based on risk:

| Change characteristics | Depth |
|---|---|
| Touches auth, payments, data deletion, crypto, or user-facing security | Maximum — read every line, trace every path |
| New feature with significant logic, new API surface, or schema migration | High — full Layer 0-3 review |
| Bug fix with clear scope and existing test coverage | Medium — verify fix correctness, check for similar bugs elsewhere, verify test |
| Config change, dependency bump, documentation update | Light — verify intent, check for obvious errors |
| Generated code, formatting-only changes, trivial renames | Scan — verify the change is what it claims to be |

---

## Severity Calibration

| Severity | What qualifies | Default action |
|---|---|---|
| Critical | Data loss, security hole, broken existing functionality, blocking regression, wrong direction (Layer 0/1 problems that invalidate the change) | Must fix before merge |
| Important | Missing error handling, test gaps, unmet requirements, design concerns that don't invalidate but weaken the change, unaddressed caller impact | Should fix; defer only with written justification |
| Minor | Style, naming, optimization, documentation improvements, subjective preferences | Author decides |

**Layer 0/1 problems are almost always Critical or Important.** A change that degrades system health or solves the wrong problem is not a Minor issue.

---

## Remote PR vs Local Changes

- For remote PRs: read the PR description and any existing review comments before starting. Understanding intent prevents misread findings.
- For local changes: `git diff` for unstaged, `git diff --staged` for staged. Clarify scope with the author if it is ambiguous.

## Automated Checks

Run the project's preflight, lint, or test suite when the change is substantial or when test coverage is in question. A passing review with failing automated checks is not a passing review. Skip only when infeasible (no test runner configured, environment not available) — if skipped, say so explicitly in the review.

---

## Failure Signals

Stop and reassess if:

- you are reviewing implementation details before understanding what the change is supposed to do — go back to Layer 0
- you have not asked "should this change exist?" and "is this the right approach?" — you skipped the highest-value layers
- you are reviewing diff hunks without having read the surrounding context for each changed function
- you are about to write "looks good" or approve without having read the code
- every finding is labeled Critical — severity inflation makes all feedback untrustworthy
- all your findings are Layer 3 (style, naming, minor cleanups) and you found nothing at Layers 0-2 — either the change is genuinely excellent, or you didn't look hard enough at the higher layers
- findings lack file:line references — specificity is the baseline for actionable feedback
- the review ends without a verdict — an open-ended summary is not a review
- you are giving feedback on code outside the diff — you are responsible for what you reviewed, not what you assumed
- a finding describes a vague risk without naming what breaks and why ("this could cause issues")
- you assessed correctness without checking the callers of changed functions — the call site is often where the real problem surfaces

---

## Output Format

See `templates/review-output.md` for the full output structure.

**Required sections:** Change Assessment (Layer 0-1 findings) · Strengths · Issues (by severity) · Verdict

**For each issue:**
- Layer reference (which layer this finding belongs to)
- `File:line` reference
- What is wrong
- Why it matters
- How to fix (if not obvious)

**Verdict options:** `Ready to merge` / `Not ready` / `Ready with fixes`

---

## Self-Check Before Exiting

- [ ] Did I understand the intent of the change before reviewing the implementation?
- [ ] Did I ask "should this change exist?" and "is this the right approach?" (Layers 0-1)?
- [ ] Did I read enough context around each changed function (not just the diff hunks)?
- [ ] Did I check the callers of changed functions for breakage?
- [ ] Does every issue have: file:line, what is wrong, why it matters?
- [ ] Is my severity calibration honest (not inflated, not deflated)?
- [ ] Did I give a clear verdict (Ready / Not ready / Ready with fixes)?
- [ ] Am I exiting because the review is genuinely complete, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
