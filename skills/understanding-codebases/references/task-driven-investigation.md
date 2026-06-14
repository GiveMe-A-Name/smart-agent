# Task-Driven Working Models

Different tasks need different working-model slices before implementation. The sections below name the minimum relationships to establish, not a fixed sequence. Skip a slice only when repository evidence already establishes it; add a slice when a lookup reveals an unresolved caller, data source, override, convention, or verification gate.

## Fixing a Bug

Before editing, name these working-model slices:
- **Symptom boundary**: the error message, wrong output, failing test, or user-visible behavior to change.
- **Reproduction path**: the entry point, test path, log path, or operation that can produce the symptom.
- **Divergence point**: where actual behavior first differs from expected behavior.
- **Data/config contributors**: inputs, state, fixtures, defaults, flags, or config values that can produce the same symptom.
- **Affected consumers**: callers or users that rely on the current behavior, including those that may depend on the bug if the behavior is longstanding.
- **Verification path**: the test that already fails, the missing test that would fail before the fix, or the smallest new test that would fail before the fix.

## Adding a Feature

Before editing, name these working-model slices:
- **Analogue**: at least one similar feature, unless search shows none exists.
- **Convention**: file location, naming, registration, error handling, logging, validation, fixture, and test patterns used by the analogue.
- **Extension point**: the registry, router, schema, provider, plugin point, base class, or generator where the new feature plugs in.
- **Data path**: how data enters, changes shape, and reaches the new behavior.
- **Shared utilities**: helpers, base classes, test builders, or generators used by the analogue and applicable to the new feature.
- **Discovery gates**: build, package, generated-code, lint, fixture, or test configuration that controls whether new code is included.
- **Verification path**: tests, fixtures, snapshots, generated files, or docs that analogous features update.

## Understanding for Refactoring

Before editing, name these working-model slices:
- **Current contract**: inputs, outputs, exceptions, side effects, ordering, and persistence behavior callers rely on.
- **Consumer set**: direct callers, implementations, or references affected by the refactor.
- **Preserved behavior**: what must remain identical after the refactor.
- **Preparation boundary**: which edits are behavior-preserving preparation and which edit changes behavior, if any.
- **Verification path**: tests or checks that prove behavior was preserved.
- **Risk boundary**: consumers or dynamic paths not traced, and why they are outside the current edit.

## Understanding Architecture or "How Does X Work"

Before explaining or changing architecture, name these working-model slices:
- **Module boundary**: the package, service, layer, or subsystem that owns the behavior.
- **Entry path**: one concrete request, command, job, event, or library call traced end to end.
- **Contracts**: public APIs, interfaces, schemas, or persisted data shapes between layers.
- **Communication path**: how layers call each other or exchange data.
- **Intent source**: README, ADR, design doc, commit history, or tests that explain why the shape exists, if such evidence is present.
- **Change relevance**: which part of the architecture constrains the current task and which parts were intentionally left untraced.
