# Plan Review Output Template

Use this structure for every review. Omit a section only if there are genuinely no findings at that level — do not omit to appear positive.

---

## Plan Assessment

[Summarize what this plan intends to achieve in 1–2 sentences, in your own words. Then state your assessment at Layers 0–1.]

**Repo alignment (Layer 0):** [Does the plan's approach conflict with CLAUDE.md, architecture docs, or existing conventions? If there are violations at this level, state them here — they are the most important findings in the review and may block implementation regardless of plan quality.]

**Design and scope (Layer 1):** [Is this the right approach? Does the scope match what the plan claims? Are the real risks identified? Are there technical assumptions that need validation?]

## Strengths

[What is well done — be specific. Cite plan section or line number.
Examples: "Risk ordering is correct — most uncertain task is first", "Every acceptance step is an executable command with expected output", "Task boundaries are clean and each task is independently verifiable"]

## Issues

### Critical (Must Fix Before Implementation)

[Layer 0 violations, approach-invalidating flaws, technical assumptions that are false and would cause wrong behavior.
If none: omit this section.]

**1. [Short title]**
- **Layer:** [0: Repo Alignment / 1: Design & Scope / 2: Internal Consistency / 3: Executability]
- **Location:** `plan-file.md:line`
- **What:** [Specific description of what the plan does or assumes that is wrong]
- **Why:** [What breaks in production if the plan proceeds — not "may cause issues" but "will permanently skip partial-write days" or "a bug here fails every API endpoint simultaneously with no partial rollback." The constraint citation belongs in Location or What as evidence, not here: "violates AGENTS.md:42" is not a Why.]
- **Fix:** [How to address it — for Layer 0 issues this often means changing the approach, not adding a note]

### Important (Should Fix)

[Internal inconsistencies, false technical assumptions, real risks not identified, non-executable verification steps.
If none: omit this section.]

**1. [Short title]**
- **Layer:** [0 / 1 / 2 / 3]
- **Location:** `plan-file.md:line`
- **What:** [Specific description]
- **Why:** [What fails, becomes unreliable, or cannot be verified if this goes unfixed — a concrete consequence, not "may cause issues"]
- **Fix:** [Suggestion, if not obvious]

### Minor (Nice to Have)

[Missing detail that won't cause implementation problems, wording.
If none: omit this section.]

**1. [Short title]**
- **Location:** `plan-file.md:line`
- **What / Fix:** [Can be combined for brevity]

## Verdict

**Ready to implement?** `Approved` / `Approved with fixes` / `Blocked`

**Reasoning:** [One or two sentences of technical justification. Name which layer and which specific issues drive the verdict. If No, state what must change before re-review.]

---

## Example

### Plan Assessment

This plan builds an incremental factor computation pipeline that reads price data and writes factor values to a ClickHouse table.

**Repo alignment (Layer 0):** The plan introduces `factor_builder/cli.py` and a `python -m factor_builder build` entry point. This conflicts with the repository's constraint at AGENTS.md:42 requiring all batch jobs to reuse `build_range_cli`. The plan's CLI shape is incompatible with the established pattern.

**Design and scope (Layer 1):** The incremental completeness check operates at the day level, but the storage primary key is `(factor_name, trade_date, ts_code)`. A partial write that succeeds for some stocks will mark the entire day as complete, permanently skipping the missing rows. This is a fundamental data-integrity flaw, not an implementation detail.

### Strengths
- Risk ordering is correct — the most uncertain integration point is tackled first (Task 1)
- Every acceptance step is an executable command with expected output

### Issues

#### Critical

**1. CLI architecture violates repo convention**
- **Layer:** 0: Repo Alignment
- **Location:** `docs/plans/2026-04-21-factor-builder.md:83`, `AGENTS.md:42`
- **What:** The plan introduces a standalone `factor_builder/cli.py` and `python -m factor_builder build` entry point. (AGENTS.md:42 requires all batch jobs to use `build_range_cli`.)
- **Why:** This job will be invisible to the scheduler — it cannot be triggered, monitored, or retried by ops tooling. Every other builder in the repo participates in scheduling and alerting; this one runs silently outside that infrastructure. Failures go undetected.
- **Fix:** Replace with a `build_range_cli` subclass. Remove the standalone `factor_builder/__main__.py`.

**2. Incremental completeness check has partial-write blind spot**
- **Layer:** 1: Design & Scope
- **Location:** `docs/plans/2026-04-21-factor-builder.md:67`, `docs/plans/2026-04-21-factor-builder.md:155`
- **What:** `get_existing_dates()` marks a day complete if any record exists, but the primary key is `(factor_name, trade_date, ts_code)`.
- **Why:** After a partial write, re-runs will permanently skip the missing stocks while reporting success. The plan encodes `dates_computed = 0` as the re-run success criterion, which hardens this defect into the acceptance conditions.
- **Fix:** Change completeness check to operate on `(trade_date, ts_code)` pairs, or perform idempotent cleanup before each write.

### Verdict

**Ready to implement?** Blocked

**Reasoning:** The CLI architecture violates a hard repo constraint (Critical, Layer 0), and the incremental completeness check has a fundamental data-integrity flaw (Critical, Layer 1). Both require approach changes, not implementation fixes, before this plan can be executed.
