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
  - the actual operating environment and supported usage relevant to the change;
  - the semantic center and current owner for each material behavior delta;
  - the behavior or contract delta;
  - affected callers, consumers, state, side effects, or operational boundaries relevant to the change;
  - material unknowns that could change the assessment.
- Open the primary changed implementation and enough surrounding code to explain how the changed behavior is reached. Do not form the model from the description, file list, hunk, symbol name, or test alone.
- If intent evidence is absent or conflicts with the implementation, label intent as unknown or conflicting. Do not invent product priorities, architectural direction, or author motivation.
- Ground operating assumptions in the request, callers, configuration, and deployment evidence. Database access does not imply an online service, competing writers, multiple tenants, or distributed deployment. Missing context does not justify assuming the most dangerous environment; identify only unknowns that could materially change this review.

### Design Gate

- Before judging local implementation details, identify the repository component currently making each affected decision and compare the new placement with the dependency, state, and contract boundaries exercised by the change.
- A design concern must cite the repository evidence that establishes the current owner, boundary, convention, migration direction, or simpler supported mechanism. A different design preference is not a defect.
- Review the necessity of new defenses, abstractions, dependencies, and state against current use. A plausible unusual operation or hypothetical future deployment does not establish a requirement to support it. Consider whether a clear failure or user retry already meets the product's needs before accepting recovery or coordination machinery. Preserve inexpensive checks with concrete benefits; do not invent occurrence rates or reject a check solely because its trigger is rare.
- Unnecessary complexity can require changes even when functionality and tests pass. Identify the unsupported premise, concrete maintenance burden, and simpler way to preserve required behavior. Substantial extra state, dependencies, or duplicated policy can justify blocking approval; minor redundancy remains non-blocking.
- If the change alters a public interface, exported symbol, API, event, schema, config, error behavior, side effect, ordering, identity, or persistent state, search for repository-visible callers and consumers, inspect those that can observe the delta, and state external or uninspected consumers as coverage limits before approval.
- If a design flaw would replace or invalidate substantial implementation work, report it before lower-impact findings.

### Test Judgment

When tests are added or modified, review them by the behavior they check, in this order. Keep the comparison scoped to the affected behavior; a separate review round, reviewer, or per-test report is not required.

1. **Is the behavior worth protecting?** Connect it to a current requirement, supported workflow, reasonable everyday mistake, or relevant observed failure. Identify what would matter to a user or caller if it broke. A constructible scenario or an existing code branch does not establish that the behavior deserves support.
2. **Is automated testing worth the cost?** Judge the expected value against setup, execution, and ongoing maintenance, including mocks and coupling to implementation details. Existing checks or direct verification may be sufficient; follow repository requirements. A test can be unique and still not worth adding.
3. **Does the test actually verify that behavior?** Inspect its setup and assertions to identify which meaningful error would make it fail. Testing generated source text may verify a code-generation contract, but does not by itself prove the generated code's runtime behavior. User-oriented goals may be verified at a unit or integration boundary; they do not require end-to-end tests by default.
4. **Does existing verification already do the job?** Inspect relevant existing tests and compare what errors they can detect. Explain the added protection or other concrete benefit, such as easier failure diagnosis, before retaining overlapping tests. Sharing a code path does not alone make tests redundant; different inputs may protect different caller contracts.

Review a mechanism and its tests together: testing invented behavior does not justify that behavior. Recommend removing or revising tests when their value does not warrant their cost, while preserving required behavior. Apply the same reasoning before requesting a missing test; code changes alone do not justify one.

Before finishing a review with changed tests, give a brief evidence-backed judgment in Verification and Coverage, naming the relevant behavior and tests. If necessity, effectiveness, or comparison with existing checks was not assessed, mark that part as not evaluated. Passing tests do not complete this assessment, and finding no unnecessary tests is a valid result.

### Risk Gate

- Every finding must identify the changed decision, concrete consequence, and supporting file/line or check. For behavioral defects, show the reachable condition under which they fail. For unnecessary implementation or tests, show the unsupported scope or redundant protection and the specific maintenance activity, dependency, or state burden it introduces; do not invent a runtime failure to justify it. Omit claims whose consequence cannot be grounded.
- Do not report unrelated defects outside the diff unless the change directly exercises them or violates their contract.
- If the repository exposes tests or merge gates for the touched behavior, inspect their status. Report each executed check as passed or failed and touched-area checks not executed as not run; never imply verification that did not occur.
- State coverage limits when changed files, consumers, runtime behavior, or checks were not inspected. A coverage limit is uncertainty, not evidence that a defect exists.
- Do not implement fixes during a review unless the user also asks for changes.

## Review Output

Use `templates/review-output.md` for a complete review. Keep the result proportional to the evidence and lead with the highest merge risk.

Minimum content:

- **Change Model:** intent, operating context, supported usage, semantic center, behavior or contract delta, affected boundaries, and material unknowns.
- **Design Assessment:** whether the change shape and added complexity fit actual use, and the evidence supporting that judgment.
- **Findings:** concrete issues ordered by merge risk, or an explicit statement that none were found.
- **Verification and Coverage:** checks passed, failed, or not run; when tests change, their necessity, effectiveness, and overlap with existing verification; plus material inspection limits.
- **Verdict:** include `Ready to merge`, `Ready with fixes`, or `Not ready` when the user asks for an approval or merge decision.

For each finding include the file/line, what is wrong, the triggering path or concrete maintenance burden, why it matters, and a fix direction when the evidence supports one. Omit optional polish unless the user requests it or it materially affects future defect risk or maintenance cost.

## Calibration Examples

- **Positive:** A serializer stops emitting a field. The semantic center is the response construction, but the review continues to endpoint callers and client consumers. A finding is valid when a consumer still requires the field and the review can show the resulting failure.
- **Boundary:** A PR description says only "refactor handler." The reviewer may reconstruct the behavior-preserving mechanism from code and tests, but must not claim the refactor conflicts with product direction without roadmap, issue, history, or repository evidence.
- **Failure:** A reviewer comments on names and duplicated lines before noticing that authorization moved from a server boundary into a caller-controlled branch. The review must be revised because it evaluated local polish before the behavior owner and trust boundary.
- **Unnecessary scope:** A local read-only reporting change introduces a coordinator for hypothetical competing service instances. Trace the actual entry point and deployment before judging the premise; if unsupported, identify the added lifecycle or state burden and the simpler reporting path. Passing tests do not settle that design judgment.
- **Test boundary:** A small error-message improvement has adequate existing coverage and direct verification. Do not require another test merely for the changed branch. A cheap check that improves the error is not overengineering solely because the input is uncommon.
- **Unique but unnecessary:** A local tool gains an automated recovery test for users manually changing its internal files during execution. No other test covers it, but that does not establish a supported use case or justify the recovery machinery and test maintenance. Establish the behavior's value before checking for duplication.
