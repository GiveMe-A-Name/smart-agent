---
name: understanding-codebases
description: "Build an implementation-ready working model of the relevant codebase slice. Use when implementing, fixing, refactoring, or extending code and the needed behavior, entry points, data flow, consumers, conventions, tests, or risks are not yet mapped from repository evidence."
---

# Codebase Understanding

Build the smallest task-specific working model needed before editing code. The goal is not to tour the repository or collect citations; it is to know where the change belongs, how the relevant behavior works, what local conventions constrain the shape of the change, who or what can break, and how the result can be verified. Repository evidence is the material for that model, not the final product.

## Principles

- **Treat descriptions as hypotheses, not evidence.** User explanations, issue text, PR summaries, filenames, and function names can point to where to look; they do not prove what the code does [because they reflect a human or naming model of behavior, while the implementation may diverge].
- **Investigate from the intended change, not from curiosity.** Each search or file read must reduce a concrete implementation uncertainty such as "where does this behavior enter?", "where does the decision happen?", "who consumes this contract?", "what existing pattern applies?", or "what test would fail if this breaks?" [because undirected browsing creates confidence from volume while leaving the change point, impact, or verification path unproven].
- **Trace past wrappers to behavior-changing code.** Entry points, exports, registries, adapters, middleware, and config wiring are useful waypoints, but stop only after reaching code that makes decisions, transforms data, mutates state, performs I/O, or enforces a contract relevant to the task [because wrappers prove reachability, not where a behavior change will take effect].
- **Model data and state movement before changing behavior.** For changes that depend on values, objects, records, or state transitions, trace where the data comes from, its shape, where it is transformed or persisted, and which values control branches [because editing a local expression without the surrounding data model can fix the observed case while breaking another path or leaving the real source unchanged].
- **Use local patterns before inventing a shape.** Before adding or restructuring code, find analogous implementations, tests, helpers, registration paths, and naming/file conventions [because codebase conventions define the path of least surprise for maintainers, reviewers, and test infrastructure].
- **Map consumers and contracts before changing shared code.** When a function, type, config value, event, API, or persisted shape may have multiple users, trace direct callers or consumers and the assumptions they make [because a locally correct edit can break a wider contract the edited file does not reveal].
- **Keep scope evidence-driven and change-sized.** Start from the smallest slice that can support the current edit; widen only when a read file or command result reveals an unresolved dependency, caller, override, convention, or verification gate needed for the change [because broad sweeps delay implementation while narrow untraced edits create hidden risk].
- **Separate proven, hypothesized, and unknown.** A useful working model can contain uncertainty, but it must label uncertainty explicitly and never treat an untraced path, guessed convention, or inferred caller impact as proven.

## Working Model Constraints

- Before editing behavior, identify the intended task boundary: the behavior to add, fix, remove, or preserve, and any user-stated non-goals already present in the conversation.
- Before choosing a change point, trace at least one relevant execution or data path from an entry point to the behavior-changing code, unless the edit is confined to a file whose caller and behavior context were already read in this session.
- Before editing behavior-changing code, identify the specific code that makes the relevant decision, transformation, state mutation, I/O call, or contract enforcement; do not stop at a wrapper, registry, adapter, export, or function name unless that layer itself owns the behavior.
- Before changing shared code, inspect direct callers, references, implementations, or consumers until the impacted contract is clear enough to name which consumers are in scope and which are untraced.
- Before adding a feature or new variant, inspect at least one analogous implementation and its test or registration path when such an analogue exists in the repository.
- Before changing data-dependent behavior, identify the relevant data shape and at least one source, transformation point, or persistence/access point that constrains the edit.
- Before implementation, identify the verification path: existing tests to run, missing tests to add, or the reason no local verification is available. For new code, also read build/package/test configuration when it controls discovery, generation, linting, fixtures, or test execution.
- If a value controls behavior, check whether it can be overridden by environment variable, CLI argument, config file, feature flag, test fixture, or deployment setting. Record the default and the override mechanism together.
- If a file is listed as relevant, state its role in the traced slice. A file list without relationships is not evidence.
- Stop expanding only after the working model can name the change point, relevant entry/data path, local convention or analogue, consumer/contract risk, verification path, and remaining unknowns that are not on the critical path. Continuing to read unrelated files after this point is undirected browsing.
- For pure explanation questions, use the same working-model standard but stop at the slice needed to explain behavior; name any untraced branches that could change the explanation.

See `references/working-model-slices.md` when deciding which parts of the implementation working model are required for the current task.
See `references/investigation-techniques.md` when choosing how to trace entry points, delegates, consumers, patterns, types, tests, history, or directory structure.
See `references/task-driven-investigation.md` when the task type is clear but the starting strategy is not.
See `references/challenging-situations.md` when the codebase is large, infrastructure-heavy, poorly named, or poorly documented.

Example index:

| Example | Read when |
|---------|-----------|
| `examples/tracing-behavior.md` | The current evidence stops at an entry point, wrapper, facade, or function name and behavior-changing code has not been reached. |
| `examples/pattern-discovery.md` | The task is action-oriented and you need to identify existing conventions or extension points before implementing a change. |
| `examples/bug-fix-working-model.md` | A bug fix depends on data shape, consumers, config, or verification rather than a single obvious line edit. |
