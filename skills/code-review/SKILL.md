---
name: code-review
description: "Evaluate code changes by reconstructing their intent, behavior, affected contracts, and merge risk. Use when a diff, staged or local changes, or a PR exists and the user asks to review, audit, approve, or judge those changes."
---

# Code Review

Build an evidence-backed model of the change before producing findings. A review is not a scan of changed lines: first understand what behavior is meant to change, locate the logic that makes that behavior happen, and trace what the decision can affect. Use that model to judge design and merge risk before examining local polish.

## Principles

- **Review intent, mechanism, and contracts together.** Reconstruct the intended behavior, determine how the implementation changes it, and compare that result with existing caller-visible contracts [because a plausible implementation can solve the wrong problem or break behavior outside the diff].
- **Navigate from the change outward.** Start with the overall purpose and scope, inspect the semantic center next, then follow relevant callers, consumers, state, side effects, and verification before reviewing the remaining files [because the important review order follows behavior and risk, not filename or diff order].
- **Find semantic centers, not the largest hunks.** For each independent behavior delta, prioritize the branch, state transition, boundary, contract, or orchestration point that decides it; do not force unrelated behavior paths into one model [because a design error at a semantic center invalidates otherwise-correct supporting edits].
- **Derive risks from concrete impact paths.** Trace a changed decision to a reachable input, caller, consumer, state, deployment condition, or failure mode and then to an observable consequence [because generic checklists produce hypothetical warnings without showing that the change can cause them].
- **Treat descriptions as hypotheses and tests as evidence.** PR text, issues, commit messages, names, and comments help recover intent but do not prove behavior; tests help only when their assertions would fail if the intended contract broke.
- **Spend review attention by consequence.** Surface wrong direction, misplaced responsibility, broken contracts, unsafe state or rollout, and behavioral defects before maintainability or style [because local polish must not hide a change that should be redesigned or must not merge].

## Constraints

### Understanding Gate

- Do not review without changed code to inspect. A review request without a diff, local or staged changes, or a PR is missing its subject.
- Before recording findings, establish a change model from repository evidence containing:
  - the intended observable behavior and its evidence source;
  - the semantic center and current owner for each material behavior delta;
  - the behavior or contract delta;
  - affected callers, consumers, state, side effects, or operational boundaries relevant to the change;
  - material unknowns that could change the assessment.
- Open the primary changed implementation and enough surrounding code to explain how the changed behavior is reached. Do not form the model from the description, file list, hunk, symbol name, or test alone.
- If intent evidence is absent or conflicts with the implementation, label intent as unknown or conflicting. Do not invent product priorities, architectural direction, or author motivation.

### Design Gate

- Before judging local implementation details, identify the repository component currently making each affected decision and compare the new placement with the dependency, state, and contract boundaries exercised by the change.
- A design concern must cite the repository evidence that establishes the current owner, boundary, convention, migration direction, or simpler supported mechanism. A different design preference is not a defect.
- If the change alters a public interface, exported symbol, API, event, schema, config, error behavior, side effect, ordering, identity, or persistent state, search for repository-visible callers and consumers, inspect those that can observe the delta, and state external or uninspected consumers as coverage limits before approval.
- If a design flaw would replace or invalidate substantial implementation work, report it before lower-impact findings.

### Risk Gate

- Every finding must identify the changed decision, the reachable condition under which it fails, the observable consequence, and the supporting file/line or check. Omit claims whose path or consequence cannot be grounded.
- Do not report unrelated defects outside the diff unless the change directly exercises them or violates their contract.
- If the repository exposes tests or merge gates for the touched behavior, inspect their status. Report each executed check as passed or failed and touched-area checks not executed as not run; never imply verification that did not occur.
- State coverage limits when changed files, consumers, runtime behavior, or checks were not inspected. A coverage limit is uncertainty, not evidence that a defect exists.
- Do not implement fixes during a review unless the user also asks for changes.

## Review Output

Use `templates/review-output.md` for a complete review. Keep the result proportional to the evidence and lead with the highest merge risk.

Minimum content:

- **Change Model:** intent, semantic center, behavior or contract delta, affected boundaries, and material unknowns.
- **Design Assessment:** whether the change shape is sound and the evidence supporting that judgment.
- **Findings:** concrete issues ordered by merge risk, or an explicit statement that none were found.
- **Verification and Coverage:** checks passed, failed, or not run, plus material inspection limits.
- **Verdict:** include `Ready to merge`, `Ready with fixes`, or `Not ready` when the user asks for an approval or merge decision.

For each finding include the file/line, what fails, the triggering path or condition, why it matters, and a fix direction when the evidence supports one. Omit optional polish unless the user requests it or it materially affects future defect risk.

## Calibration Examples

- **Positive:** A serializer stops emitting a field. The semantic center is the response construction, but the review continues to endpoint callers and client consumers. A finding is valid when a consumer still requires the field and the review can show the resulting failure.
- **Boundary:** A PR description says only "refactor handler." The reviewer may reconstruct the behavior-preserving mechanism from code and tests, but must not claim the refactor conflicts with product direction without roadmap, issue, history, or repository evidence.
- **Failure:** A reviewer comments on names and duplicated lines before noticing that authorization moved from a server boundary into a caller-controlled branch. The review must be revised because it evaluated local polish before the behavior owner and trust boundary.
