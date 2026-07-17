# Evaluation Criteria

Use the relevant dimensions when a feedback item's value to the current change is unclear. This is a judgment lens, not a fixed sequence.

## Intent and Risk Relevance

Determine whether the feedback affects what the change must achieve or preserve. Relevant effects include caller-visible behavior, contracts, security, data integrity, compatibility, verification, and material regression risk.

An issue may be real without being relevant to the current change. Distinguish current-change risk from pre-existing debt, adjacent cleanup, stylistic preference, and hypothetical future needs. Independently material security, data-loss, crash, or correctness risks must still be surfaced.

## Consequence Test

Ask: **what materially happens if this is not addressed now?**

- Incorrect behavior, broken contract, security exposure, data loss, crash, or measurable degradation is consequential.
- A maintenance concern is consequential only when the affected code, callers, or near-term change path makes the cost concrete.
- Aesthetic preference, abstract purity, or unsupported future flexibility has no established consequence.

If the consequence cannot be made concrete from available evidence, do not promote the item merely because the suggestion sounds reasonable.

## Evidence Test

Read the commented code and inspect the evidence needed by the claim:

- callers and tests for reuse or testability claims;
- observable behavior and contracts for correctness claims;
- supported environments for compatibility claims;
- issue, PR, conversation, or prior decisions when the judgment depends on intent.

Do not require explicit intent when repository behavior and constraints already resolve the decision. Clarify only when missing intent would materially change the disposition.

## Proportionality Test

Compare concrete benefit and risk reduction against churn, regression risk, abstraction, indirection, and maintenance cost. Prefer the response that protects the current intent without speculative generality.

Signals of negative net value include:

- an abstraction without a current shared rule or consumer;
- future-proofing without a stated requirement or extension point;
- moving local behavior across layers without reducing current risk;
- expanding the change for architectural neatness;
- a remedy whose complexity exceeds the consequence it prevents.

## Disposition Test

- **Address now** when the item is relevant, consequential, and proportionate.
- **Escalate** when it may be material but requires an unauthorized intent, scope, or architecture decision.
- **Advisory** when it is real and potentially useful later but should not affect the current change.
- **Decline** when it lacks evidence, material consequence, relevance, or positive net value.
