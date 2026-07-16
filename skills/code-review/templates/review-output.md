# Code Review Output

## Change Model

- **Intent:** [Observable behavior the change is meant to produce and the evidence source.]
- **Semantic center(s):** [Decision, state transition, boundary, contract, or orchestration point that makes each material behavior delta happen.]
- **Behavior/contract delta:** [What becomes observably different.]
- **Affected boundaries:** [Relevant callers, consumers, state, side effects, deployment, or operational surfaces inspected.]
- **Unknowns:** [Missing evidence that could materially change the assessment, or `None`.]

## Design Assessment

[State whether responsibility placement and affected boundaries are sound. Cite the code, contract, convention, or history that supports the conclusion.]

## Findings

[Order findings by merge risk. If there are none, say so explicitly.]

### [Finding title]

- **File:** `path/to/file:line`
- **Failure:** [Changed decision and what is wrong.]
- **Trigger:** [Reachable input, state, caller, consumer, or operational condition.]
- **Consequence:** [Observable correctness, security, data, compatibility, operability, or maintainability impact.]
- **Direction:** [Supported fix direction, when clear.]

## Verification and Coverage

- [Check]: [Passed / Failed / Not run]
- **Coverage limits:** [Uninspected files, consumers, runtime behavior, or checks.]

## Verdict

[`Ready to merge` / `Ready with fixes` / `Not ready`, when requested, followed by the decisive reason.]
