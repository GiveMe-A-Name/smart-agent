---
name: code-review
description: "Evaluate code changes for correctness, design, and risk. TRIGGER when: a diff, staged changes, or PR exists and the task is to review them. DO NOT TRIGGER when: responding to existing review feedback (use receiving-code-review) or no code changes exist yet."
---

# Code Review

A review that catches nothing is worse than no review — it creates false confidence. The highest-value findings are not bugs in the implementation — they are problems with the change itself: wrong direction, wrong abstraction, wrong scope.

## Trigger Logic

**Invocation default**: Skipping structured review before merge lets problems compound. The cost of unnecessary rigor is a short overhead. The cost of missing it is bugs in production or findings that block merge after the fact.

Use when:
- code changes exist (local diff, staged changes, or remote PR) and the task is to evaluate them
- completing a task in subagent-driven development and review is the gate before the next step

Do not use when:
- the task is to respond to review feedback that already exists, not to evaluate new changes
- no diff or changed code exists to evaluate yet — if you are writing code, invoke this skill after the code exists

## Capability Boundary

This skill owns: reading code changes, identifying issues across all layers (justification through implementation), calibrating severity, and delivering a structured verdict.

It does NOT:
- implement fixes or refactor reviewed code
- verify that previously given feedback was applied — that belongs to the next review cycle
- claim coverage over code that was not read

## Invariants

**Upper-layer problems outweigh lower-layer problems.** A change that goes in the wrong direction does not become acceptable because the implementation is clean. Always work from the top layer down — if a higher-layer problem is severe enough, stop and surface it immediately rather than cataloging line-level issues in code that may need to be rewritten.

**Read context, not just diff hunks.** For every changed function or module, read enough surrounding code to understand what callers expect, what invariants the module maintains, and what the changed lines actually do in context.

**Only give feedback on code you actually read.** Never approve or comment on code outside the diff or files explicitly opened. "Looks good" without reading is a false safety signal.

**Every issue needs: file:line, what is wrong, why it matters.** Vague findings ("improve error handling", "this could be cleaner") are not actionable.

**Severity must reflect actual risk.** Critical means data loss, security vulnerability, broken functionality, or blocking regression — not style, not preference. Inflating severity makes all feedback less trustworthy.

**Run the project's automated checks before approving.** A passing review with failing lint/tests is not a passing review. If checks were skipped, say so explicitly.

**Give a clear verdict.** Every review ends with: Ready / Not ready / Ready with fixes. An ambiguous conclusion defers the merge decision to the author, which is the reviewer's job.

## Failure Signals

Stop and reassess if:

- you are reviewing implementation before establishing what the change is supposed to do — "should this change exist?" and "is this the right approach?" must be answered before line-level review
- you are reviewing diff hunks without having read the surrounding context for each changed function
- you assessed correctness without checking the callers of changed functions — the call site is often where the real problem surfaces
- you are about to write "looks good" or approve without having read the code
- every finding is labeled Critical — severity inflation makes all feedback untrustworthy
- all your findings are Layer 3 and you found nothing at Layers 0-2 — either the change is genuinely excellent, or you didn't look hard enough at the higher layers
- findings lack file:line, what is wrong, and why it matters — vague references and vague risks ("this could cause issues") are both non-actionable
- the review ends without a verdict — an open-ended summary is not a review
- you are giving feedback on code outside the diff — you are responsible for what you reviewed, not what you assumed

## Completion Criteria

- [ ] The review covers justification (should this change exist?), approach (right solution?), and implementation — not just implementation details.
- [ ] Enough context was read around each changed function — including callers — not just the diff hunks.
- [ ] Every issue includes file:line, what is wrong, and why it matters, with honest severity calibration.
- [ ] Relevant specialized lenses (Security, Performance, Migration, Dependency, API Design) were applied where applicable — see `references/specialized-lenses.md`.
- [ ] User-defined design judgment dimensions were applied where relevant — see `references/user-review-standards.md`.
- [ ] If the code appears AI-generated, reviewer-specific checks were applied — see `references/ai-generated-code.md`.
- [ ] A clear verdict is present (Ready to merge / Not ready / Ready with fixes).

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the review is real.

Did I mistake superficial diff reading for sufficient context?

Am I exiting because the review is genuinely complete, or because the current findings list looks complete enough?

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

See `templates/review-output.md` for the full output structure.

**Required sections:** Change Assessment (Layer 0-1 findings) · Strengths · Issues (by severity) · Verdict

**For each issue:**
- Layer reference
- `File:line` reference
- What is wrong / Why it matters / How to fix (if not obvious)

**Verdict options:** `Ready to merge` / `Not ready` / `Ready with fixes`
