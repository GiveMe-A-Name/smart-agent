# Code Review Output Template

Use this structure for every review. Omit a severity section only if there are genuinely no findings at that level — do not omit to appear positive.

---

## Strengths

[What is well done — be specific. Cite file:line or name the pattern.
Examples: "Clean error propagation in auth.ts:42-58", "Tests cover the failure path at handler.test.ts:91"]

## Issues

### Critical (Must Fix Before Merge)

[Bugs that corrupt data, break existing functionality, or introduce security vulnerabilities.
If none: omit this section.]

**1. [Short title]**
- **File:** `path/to/file.ts:line`
- **What:** [Specific description of the problem]
- **Why:** [What breaks, what risk is introduced]
- **Fix:** [How to address it, if not obvious]

### Important (Should Fix)

[Architecture problems, missing error handling, unmet requirements, test gaps.
If none: omit this section.]

**1. [Short title]**
- **File:** `path/to/file.ts:line`
- **What:** [Specific description]
- **Why:** [Impact on correctness, maintainability, or reliability]
- **Fix:** [Suggestion, if not obvious]

### Minor (Nice to Have)

[Style, naming, documentation, optimization opportunities.
If none: omit this section.]

**1. [Short title]**
- **File:** `path/to/file.ts:line`
- **What / Why / Fix:** [Can be combined for brevity]

## Assessment

**Ready to merge?** `Yes` / `No` / `With fixes`

**Reasoning:** [One or two sentences of technical justification. Name which issues drive the verdict.]

---

## Example

### Strengths
- Clean schema migration with proper rollback (db/migrate.ts:15-42)
- Comprehensive test coverage: 18 cases including all error paths
- Error boundary with useful fallback messages (summarizer.ts:85-92)

### Issues

#### Important

**1. Missing date validation**
- **File:** `search.ts:25-27`
- **What:** Invalid ISO dates are passed directly to the query without validation
- **Why:** Silently returns no results; users see empty output with no error signal
- **Fix:** Validate ISO format before query; throw with an example format string

**2. No --help flag in CLI**
- **File:** `index.ts:1-31`
- **What:** No `--help` case exists; `--concurrency` flag is undiscoverable
- **Why:** Users cannot self-serve; support burden increases
- **Fix:** Add `--help` with usage examples and flag descriptions

#### Minor

**1. Magic number for progress reporting interval**
- **File:** `indexer.ts:130`
- **What:** `100` is hardcoded with no named constant
- **Why:** Low discoverability; semantics are unclear at read time

### Assessment

**Ready to merge?** With fixes

**Reasoning:** Core implementation is solid with good architecture and test coverage. The date validation and help flag gaps are Important but straightforward to fix and do not affect core functionality.
