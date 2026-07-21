---
name: systematic-debugging
description: "Prevent premature fixes and false completion during debugging. TRIGGER when an observed failure or unexpected behavior exists and its cause or proposed fix is not yet evidence-backed. DO NOT TRIGGER for feature work or refactoring with no current failure."
---

# Systematic Debugging

Use evidence to gate two transitions:

```text
investigation --[supported causal explanation]--> production fix
production fix --[original failure verified]----> complete
```

## Ground the Failure

Before proposing a production fix, establish the command or action that triggers the failure, the expected behavior, the actual behavior, and the first concrete mismatch. Do not substitute a nearby error or easier reproduction without stating why it is equivalent.

## Analyze Before Fixing

Inspect enough of the failing path and actual state to identify the behavior owner or external condition and a mechanism that predicts the observed symptom. Do not infer ownership from the assertion, symptom, or nearest stack-trace frame alone.

When a value or state becomes incorrect across steps or component boundaries, locate the last observed-correct boundary and the first observed-incorrect boundary. This is a conditional technique, not a required shape for every failure.

Turn the analysis into a falsifiable causal hypothesis: if the proposed mechanism is responsible, name the observation that should follow; also name an observation that would rule it out when that distinction is not already obvious. Gather evidence that discriminates between plausible explanations.

## Separate Diagnosis from Repair

Diagnostic changes such as instrumentation, logging, probes, or a minimal reproduction are allowed before the cause is confirmed. A production fix is a change intended to remain.

Do not make or present a production fix until evidence supports:

- the specific code location or external condition that owns the behavior;
- the mechanism connecting it to the observed failure;
- the predicted result of changing that mechanism.

If any part remains unresolved, continue diagnosis or report the uncertainty. Do not disguise a hypothesis as a root cause.

Do not weaken assertions, loosen contracts, increase timeouts, or skip tests merely to remove the symptom. Do so only when evidence shows the original expectation is wrong.

## Verify Before Claiming Completion

After the production fix, rerun the original failing command or action, or an explicitly equivalent reproduction, and record the observed result. Compilation, a nearby test, or one disappearance of the symptom is not sufficient unless it exercises the claimed mechanism.

If the original failure cannot be rerun, report the fix as unverified and state what evidence is missing. Do not claim that the bug is fixed.

Stop making speculative edits when experiments no longer produce new evidence or eliminate plausible explanations. Report the observed failure, evidence gathered, hypotheses ruled out, unresolved uncertainty, and the next useful observation.
