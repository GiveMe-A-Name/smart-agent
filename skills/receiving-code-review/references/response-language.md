# Reviewer-Facing Responses

State the disposition, the load-bearing basis, and any next decision. Keep the response proportionate to the disagreement.

## Address Now

State what changed and, when non-obvious, which risk or contract it protects.

```text
Fixed by preserving the existing error contract while validating the new input path.
```

## Escalate

Name the decision that cannot be inferred and the implementation consequence of each plausible answer.

```text
This can either remain local to the new endpoint or become a shared policy for all callers. The latter changes existing error behavior, so I need the intended scope before proceeding.
```

## Advisory

State the evidence-backed finding, why it is not part of the current change, and its concrete consequence. Do not imply that follow-up work was created.

```text
The cache has no capacity limit, but this change does not alter its lifetime or write volume, so I have not expanded the patch. Long-running processes may still need a separate capacity policy.
```

## Decline

Name the missing consequence or contradicting evidence rather than arguing from preference.

```text
I am not extracting an interface here: there is one consumer and no current extension requirement, so the change would add indirection without reducing this patch's risk.
```

Agreement language is not a substitute for evaluation. Politeness is fine; make the technical disposition and evidence clear.
