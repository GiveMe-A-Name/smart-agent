---
name: understanding-codebases
description: "Ground codebase answers and candidate change points in repository evidence. TRIGGER when answering a codebase question or preparing a code change requires behavior, ownership, patterns, call flow, or conventions not already traced in repository evidence."
---

# Codebase Understanding

Build a task-specific repository slice before answering codebase questions or suggesting changes. The goal is not to tour the repository; it is to replace guesses about code behavior with cited evidence from files, command results, and traced call paths.

## Principles

- **Treat descriptions as hypotheses, not evidence.** User explanations, issue text, PR summaries, filenames, and function names can point to where to look; they do not prove what the code does [because they reflect a human or naming model of behavior, while the implementation may diverge].
- **Investigate from a question, not from curiosity.** Each search or file read must answer a current hypothesis such as "where is the entry point?", "where does this delegate?", "who calls this?", or "how does the codebase do similar work?" [because undirected browsing expands context without closing the evidence gap].
- **Trace past wrappers to behavior-changing code.** Entry points, exports, registries, adapters, middleware, and config wiring are useful waypoints, but stop only after reaching code that makes decisions, transforms data, mutates state, performs I/O, or enforces a contract [because wrappers prove reachability, not behavior or change ownership].
- **Use local patterns before proposing a new shape.** For action-oriented questions, find analogous implementations, tests, helpers, and registration paths before naming a candidate change point [because codebase conventions define the path of least surprise for maintainers and tests].
- **Keep scope evidence-driven.** Start from the smallest slice that can answer the question; widen only when a read file or command result reveals an unresolved dependency, caller, override, or convention needed for the answer [because broad sweeps create confidence from volume rather than relevance].
- **Separate proven, hypothesized, and unknown.** A useful answer can contain uncertainty, but it must label uncertainty explicitly and never promote an untraced path into a conclusion.

## Investigation Constraints

- Before answering, every factual conclusion about behavior, ownership, convention, or change location must point to at least one source file, line, command result, or traced call path already read in this session.
- If the task is a pure explanation question, trace the relevant call/data flow far enough to explain behavior and name any untraced branches that could change the answer.
- If the task is action-oriented ("add X", "fix X", "replicate X", "where should this change go"), identify candidate change points and state why each is supported by behavior, callers, or existing conventions.
- If a file is listed as relevant, state its role in the traced slice. A file list without relationships is not evidence.
- If a value controls behavior, check whether it can be overridden by environment variable, CLI argument, config file, feature flag, test fixture, or deployment setting. Report the default and the override mechanism together.
- For add-a-feature investigations, read build/package configuration and test infrastructure when they define package discovery, linting, generated code, fixture patterns, or test execution. Source-only evidence is insufficient when new code must pass those gates.
- Stop expanding after the current evidence answers the task and the remaining unknowns are not on the answer's critical path. Continuing to read unrelated files after this point is undirected browsing.

See `references/investigation-techniques.md` when choosing how to trace entry points, delegates, consumers, patterns, types, tests, history, or directory structure.
See `references/task-driven-investigation.md` when the task type is clear but the starting strategy is not.
See `references/challenging-situations.md` when the codebase is large, infrastructure-heavy, poorly named, or poorly documented.

Example index:

| Example | Read when |
|---------|-----------|
| `examples/tracing-behavior.md` | The current evidence stops at an entry point, wrapper, facade, or function name and behavior-changing code has not been reached. |
| `examples/pattern-discovery.md` | The task is action-oriented and you need to identify existing conventions or extension points before proposing a change. |
