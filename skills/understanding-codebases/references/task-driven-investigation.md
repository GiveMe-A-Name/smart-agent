# Task-Driven Investigation

Use the task type to prioritize evidence. These are prompts, not mandatory checklists.

## Bug or unexpected behavior

Establish the symptom and expected contract, reproduce or trace one failing path, find the first divergence, and account for the data, configuration, state, and consumers that could produce the same symptom. Prefer a failing check or focused reproduction as the verification surface.

## Feature or extension

Trace one analogous feature when available: its owner, registration or discovery mechanism, data path, contracts, and tests. Record whether the pattern is an enforced mechanism, a repeated convention, or merely one example. If no analogue exists, fall back to module boundaries and explicit contracts.

## Refactoring context

Map the current contract, callers, side effects, ordering, state ownership, and verification that exposes preserved behavior. Identify where responsibilities currently live and any demonstrated boundary tension. Do not decide whether or how to refactor.

## Architecture or explanation

Trace one representative request, command, event, or library call across module boundaries. Explain the current structure, runtime collaboration, key contracts, and supported design intent. Bound variants not traced.

## When design intent matters

Consult intent evidence when the structure seems deliberate, surprising, historically constrained, or relevant to a proposed responsibility boundary. Prefer ADRs and design docs, then tests and schemas, then change history and comments. Label intent as documented, inferred, conflicting, or unknown; do not turn a plausible story into fact.
