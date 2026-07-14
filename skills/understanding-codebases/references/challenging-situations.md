# Challenging Situations

## Large codebase

Start with the narrowest plausible module and one concrete runtime path. Expand only when an unresolved dependency, consumer, data source, or contract could change the model. Stop at stable boundaries and record what remains untraced.

## Poor names or documentation

Trace types, data, side effects, and callers instead of inferring from names. Use tests and targeted history to recover behavior and intent. Do not rename code merely to make investigation easier.

## Dynamic or generated wiring

Direct reference search is incomplete when registries, reflection, dependency injection, annotations, generated code, package metadata, or feature flags select behavior. Find the discovery mechanism and trace one registered instance from loading to invocation.

## Missing or weak tests

Use existing tests for partial contract evidence, but name uncovered behavior explicitly. Identify a reproduction, log assertion, manual check, or characterization test as the verification gap; do not report unobserved behavior as verified.

## Missing or conflicting intent

Absence of rationale is not evidence that the design was accidental. When docs, tests, code, and history disagree, report each source and its date or scope. Label the conclusion `conflicting` or `unknown` rather than resolving it through speculation.

## No analogue

Look for adjacent evidence about registration, validation, persistence, errors, and tests. If none exists, describe the explicit contracts and absence of precedent. Choosing a new structure belongs to implementation judgment.
