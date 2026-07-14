# Structural Judgment Dimensions

Use this reference only when the working model exposes a concrete structural choice. Start with the matching signal; do not review every dimension by default.

## Choose the intervention

| Evidence in the current change | Supported intervention |
|---|---|
| New behavior fits the existing owner and preserves its boundaries | Local change |
| A behavior-preserving move would create the seam or owner the change needs | Preparatory refactoring, then behavior change |
| Every local patch would deepen a demonstrated boundary problem | Focused structural change |
| Old and new contracts or data shapes must coexist | Staged change |
| Owner, contract, state source, or migration order is unresolved | Pause and name the missing evidence |

Judge scope conceptually, not by line count. A multi-file change can be the narrowest coherent intervention; a one-line conditional can be structurally incomplete.

## Responsibility and complexity placement

Apply this when a rule, transformation, recovery path, or state mutation could live in more than one layer.

- Name the complete concern and the context required to decide it.
- Keep transport parsing at the transport boundary, domain rules with domain context, persistence invariants with the state owner, and recovery with the caller that can choose the outcome.
- A concern may cross layers only when each layer enforces a distinct boundary-owned rule. Repeating fragments of one rule across route, service, and storage is split ownership.
- If callers must reproduce ordering, validation, caching, or recovery details to use a module correctly, compare moving that complexity behind the responsible interface with leaving it caller-owned.

Choose a local change when the existing owner already has the required context. Choose a structural correction when the only local location lacks that context or would make a generic layer depend on domain policy.

See `../examples/complexity-misplacement.md` when one concern is already spread across layers.

## Abstraction

Apply this when the change introduces shared code, adds a branch or option to an existing abstraction, or reveals repeated policy.

### Evidence for sharing

Create or retain a shared abstraction when current consumers:

- represent the same knowledge or invariant;
- change in response to the same product, security, or lifecycle event; and
- can use one contract without caller-specific branches.

A single-use function may name or decompose behavior. Do not present it as a reusable policy, framework, or extension point without current evidence of that role.

### Evidence against sharing

Keep code separate, or split an existing abstraction, when:

- similarity is syntactic but owners or change drivers differ;
- options and branches map to caller identities;
- one caller repeatedly bypasses the abstraction;
- the shared name hides materially different errors, side effects, or lifecycles; or
- the abstraction must predict hypothetical future requirements to justify itself.

Duplication is not automatically acceptable: duplicated authoritative knowledge can drift. Surface similarity is not automatically harmful: independent policies may safely look alike. Decide from ownership and change drivers, not occurrence count.

See `../examples/wrong-abstraction.md` for the separate-policy and shared-invariant contrast.

## Contracts and compatibility

Apply this when callers can observe a changed return, error, side effect, persistence result, event, ordering, timing, cache behavior, or reference identity.

- Mark each observed behavior as `preserve`, `intentionally change`, or `migrate`.
- Signature compatibility is insufficient when execution timing, error propagation, default behavior, side effects, or identity changes.
- For shared or public boundaries, prefer an additive contract or staged migration when consumers cannot move atomically.
- Do not preserve an accidental behavior silently. If the task requires changing it, name the consumer impact and verification that distinguishes the new contract.

See `../examples/implicit-contract-break.md` when a refactor changes sync/async behavior, caching, errors, or identity.

## Security and trust boundaries

Apply this when the change adds or alters an endpoint, command execution path, authentication or authorization flow, tenant boundary, secret, sensitive field, upload, redirect, deserialization path, or dependency with new privileges.

- Identify the untrusted input, authenticated actor, protected resource, and enforcement point. Authentication establishes identity; authorization must still be checked against the requested action and resource.
- Enforce security decisions on the trusted side of the boundary. Client visibility, hidden UI, caller-provided roles, and prior validation by another consumer are not authorization evidence.
- Minimize what crosses the boundary: returned fields, logged values, error detail, credentials, filesystem or network reach, and dependency permissions.
- Follow untrusted data to its security-sensitive sink. Use the repository's established validation, escaping, parameterization, path handling, and secret-storage mechanisms at the boundary that owns the risk.
- When changing shared middleware or policy, trace bypass paths and direct callers; a secure primary route does not protect alternate entry points automatically.

Pause when the trusted enforcement point, actor-to-resource authorization rule, or sensitivity of exposed data cannot be established from the working model.

## Dependencies, state, and failure

Apply this when code crosses a module or service boundary, changes persistent state, or adds a multi-step operation.

- Name the new dependency direction. Specific policy may depend on a stable primitive; a general primitive should not import volatile feature policy.
- Keep one authoritative owner for mutable state. If state is copied or cached, define which source wins and how divergence is repaired.
- For multiple persistent steps, identify the state after each failure point. Use atomicity, idempotency, compensation, or an explicit recoverable state when partial progress would otherwise be ambiguous.
- If local and remote state can advance independently, account for replay and reconciliation rather than assuming both writes succeed together.
- If deployment order, schema compatibility, or rollback changes the safe design, use an additive or staged transition instead of an atomic replacement.

## Scope and verification

The structural work is in scope when it is necessary to give the current requirement a coherent owner, contract, dependency direction, or state transition. It is adjacent debt when the current change neither exercises nor worsens it.

Keep behavior-preserving preparation distinguishable from the behavior change when combining them would obscure review or rollback. Verify preparation against the old observable contract, then verify the new behavior against the intended contract. A passing suite is not evidence if assertions, snapshots, types, or checks were weakened to match the implementation.

Stop expanding when:

- the current requirement has one explainable owner;
- shared knowledge is neither duplicated nor forced into a caller-specific abstraction;
- contract and state transitions are explicit; and
- remaining improvements are not required by the current behavior.
