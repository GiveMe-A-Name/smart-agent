# Code Review Output

## Change Model

- **Intent:** [Observable behavior the change is meant to produce and the evidence source.]
- **Operating context:** [Actual deployment, supported usage, and relevant assumptions or limits.]
- **Semantic center(s):** [Decision, state transition, boundary, contract, or orchestration point that makes each material behavior delta happen.]
- **Behavior/contract delta:** [What becomes observably different.]
- **Affected boundaries:** [Relevant callers, consumers, state, side effects, deployment, or operational surfaces inspected.]
- **Unknowns:** [Missing evidence that could materially change the assessment, or `None`.]

## Design Assessment

[State whether responsibility placement, affected boundaries, and added complexity fit actual use. Cite the code, contract, convention, or history that supports the conclusion.]

## Findings

[Order findings by merge risk. If there are none, say so explicitly.]

### [Finding title]

- **File:** `path/to/file:line`
- **Issue:** [Changed decision and what is wrong.]
- **Condition or burden:** [Reachable failure condition, or the unsupported scope or redundant protection and its concrete maintenance burden.]
- **Consequence:** [Observable correctness, security, data, compatibility, operability, or maintainability impact.]
- **Direction:** [Supported fix direction, when clear.]

## Verification and Coverage

- [Check]: [Passed / Failed / Not run]
- **Test judgment (when tests change):** [Briefly explain whether the tested behavior and automation are worth maintaining, whether assertions verify that behavior, and how relevant existing checks compare. Name the behavior and tests supporting the conclusion; mark any unassessed part as `Not evaluated`. Passing tests alone is not this judgment.]
- **Coverage limits:** [Uninspected files, consumers, runtime behavior, or checks.]

## Verdict

[`Ready to merge` / `Ready with fixes` / `Not ready`, when requested, followed by the decisive reason.]
