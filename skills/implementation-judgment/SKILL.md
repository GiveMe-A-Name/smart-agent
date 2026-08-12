---
name: implementation-judgment
description: "Choose a structurally sound change shape from an evidence-backed working model. Use before a bug fix, feature, or refactor that can affect runtime behavior, contracts, responsibilities, abstractions, or dependencies; skip confirmed non-behavioral edits."
---

# Implementation Judgment

Use the current-system working model as evidence for deciding how a behavior change should land. Do not optimize for either the smallest diff or an idealized architecture. Choose the narrowest coherent intervention that carries the requirement without degrading the structure it exercises.

## Decide Whether to Apply

Apply the rest of this skill when the working model leaves a meaningful choice about the behavior owner, observable contract, abstraction boundary, dependency direction, state or failure ownership, migration, or whether a local patch is structurally complete.

Otherwise, stop here and proceed with the established local change. Do not run the full judgment checks when one owner clearly contains the change, observable contracts are preserved, and no new abstraction, cross-boundary dependency, persistent-state transition, or migration is introduced. Judge this gate by structural uncertainty, not task size or line count.

## Establish the Decision

Before designing the change, establish:

- the requested observable result and acceptance evidence;
- the current behavior owner and the context it has;
- caller-visible behavior that must be preserved, intentionally changed, or migrated; and
- explicit non-goals and adjacent behavior that remains out of scope.

Clarify the requirement when materially different outcomes satisfy its wording or no observable result distinguishes success. Pause when missing evidence about the owner, contract, state source, or migration order could change the design; name that evidence instead of restarting a broad investigation.

## Choose One Intervention

- **Local change:** the behavior fits the existing owner and preserves its boundaries.
- **Preparatory refactoring:** a bounded, behavior-preserving move can create the seam or owner needed by the behavior change. Verify the preparation against the old contract before adding new behavior.
- **Focused structural change:** every local patch would deepen a demonstrated ownership, abstraction, contract, dependency, or state problem. Limit the correction to the boundary exercised by the requirement.
- **Staged change:** old and new contracts, data shapes, deployments, or rollback paths must coexist during migration.
- **Pause:** the evidence needed to choose safely is unresolved.

Judge scope by coherence, not line count. A multi-file change can be the narrowest valid intervention; a one-line conditional can be structurally incomplete.

## Check the Change Shape

Use this as a required checking skeleton, not a formula with predetermined answers. Spend detail only on dimensions the change actually exercises.

### Ownership and complexity

Put a complete concern where the full decision can be made. Keep transport parsing at the transport boundary, domain policy with domain context, persistence invariants with the state owner, and recovery with the caller that can choose retry, rollback, user response, or alerting.

Crossing layers is valid when each layer enforces a distinct boundary-owned rule. It is split ownership when several layers each implement fragments of one rule. For example, in `route -> service -> storage`, keep request parsing in the route, the complete domain validation rule in the service, and storage invariants in storage.

### Abstraction

Share code when consumers represent the same knowledge or invariant, change for the same reason, and can use one contract without caller-specific branches. Keep similar code separate when owners, policies, lifecycles, or change drivers differ. If policies differ but share a mechanism, extract only the stable primitive.

Do not infer an abstraction from duplication count. A helper whose flags or branches map to caller identities is usually combining independent policies; duplicated authoritative knowledge that must change together usually needs one owner.

### Observable contracts

Treat any caller-observable return, error, side effect, persistence result, event, ordering, timing, caching behavior, or reference identity as part of the contract. Mark each affected behavior as `preserve`, `intentionally change`, or `migrate`.

Signature compatibility alone is insufficient. For example, changing a cached API from sync to async can alter error propagation, first-load concurrency, ordering, and object identity even after every caller adds `await`. Prefer an additive contract or staged migration when consumers cannot move atomically.

### Trust boundaries

When changing an entry point, authentication or authorization flow, tenant boundary, secret, sensitive field, upload, redirect, deserialization path, or privileged dependency:

- identify the untrusted input, authenticated actor, protected resource, and trusted enforcement point;
- enforce authorization on the trusted side against the requested action and resource;
- follow untrusted data to its sensitive sink using repository-established validation and escaping; and
- minimize returned fields, logged values, error detail, credentials, and filesystem or network reach.

Pause if the enforcement point, actor-to-resource rule, or data sensitivity is unknown.

### Dependencies, state, and failure

Keep dependency direction explicit: specific policy may depend on a stable primitive; a general primitive should not import volatile feature policy. Keep one authoritative owner for mutable state, and define precedence and repair when state is copied or cached.

For multi-step persistent work, identify the state after each failure point. Use atomicity, idempotency, compensation, or an explicit recoverable state where partial progress would otherwise be ambiguous. If local and remote state can advance independently, account for replay and reconciliation. Use an additive or staged transition when deployment order, schema compatibility, or rollback affects safety.

### Scope and verification

Include structural work only when the current requirement needs it for a coherent owner, contract, dependency direction, or state transition. Leave adjacent debt alone when the change neither exercises nor worsens it.

Verify caller-observable behavior at the responsible boundary. Do not weaken tests, snapshots, types, lint rules, or checks to make the design pass. Stop expanding when the requirement has one explainable owner, shared knowledge is neither duplicated nor forced together, contracts and state transitions are explicit, and the remaining improvements do not affect the requested behavior.

## Communicate the Judgment

Complete the checks internally. Report only what helps the user evaluate the change:

- the selected intervention and behavior owner;
- how affected observable contracts are preserved, changed, or migrated;
- material structural or failure risks and their verification; and
- excluded adjacent work when the boundary could otherwise be mistaken for an omission.

Explain rejected alternatives only when they are genuinely plausible. Do not turn the checking skeleton into a verbose, dimension-by-dimension report.

Read `references/performance-dimensions.md` only when the proposed shape adds I/O, work that grows with input or state, persistent resources, database access, caching, or concurrency.
