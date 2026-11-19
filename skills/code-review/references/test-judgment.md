# Test Judgment

When tests are added or modified, evaluate them by the behavior they protect, in this order. Keep the comparison scoped to the affected behavior.

1. **Is the behavior worth protecting?** Tie it to a current requirement, supported workflow, realistic mistake, or observed failure. A constructible scenario or existing branch alone does not justify support.
2. **Is automated testing worth the cost?** Weigh expected value against setup, execution, mocks, and coupling to implementation details. A test can be unique and still not worth maintaining.
3. **Does the test actually verify that behavior?** Check setup and assertions to identify which meaningful failure would break it. Testing generated source text verifies a generation contract, not runtime behavior; user-facing goals do not require end-to-end tests by default.
4. **Does existing verification already cover it?** Compare what failures existing tests catch. Retain overlapping tests only for concrete added protection or meaningfully clearer diagnosis; shared code paths alone do not make tests redundant when they protect distinct caller contracts.

Review a mechanism and its tests together: testing invented behavior does not justify the behavior. Recommend removing or revising low-value tests while preserving required behavior, and apply the same bar before requesting a missing test—code changes alone do not justify one.

State a brief evidence-backed judgment naming the relevant behavior and tests (or mark unassessed dimensions as `Not evaluated`). Passing tests do not complete this assessment, and finding no unnecessary tests is a valid result.
