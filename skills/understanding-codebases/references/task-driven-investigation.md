# Task-Driven Investigation Strategies

Different tasks have different optimal starting points. Choose the strategy that matches the task. The steps below are common starting sequences — not fixed procedures. Each step should be driven by what the previous lookup revealed, not executed mechanically in order. Skip steps that are not relevant; add steps when evidence demands it.

## Fixing a Bug

1. Start from the symptom (error message, wrong behavior, failing test)
2. Search for the symptom in code (error strings, variable names, log messages)
3. Trace the execution path that produces the symptom
4. Identify where the actual behavior diverges from expected behavior
5. Check: are there tests that should have caught this? What do they test?

## Adding a Feature

1. Find how similar features are implemented (pattern discovery)
2. Identify the conventions: file location, naming, registration, testing
3. Trace the data flow for a similar feature end to end
4. Identify the extension points where your feature plugs in
5. Check: what shared utilities or base classes should you use?

## Understanding for Refactoring

1. Map the dependency graph of the code in question (who calls what)
2. Identify all consumers of the code you plan to change
3. Understand the implicit contracts (what callers assume)
4. Look for tests that verify the current behavior
5. Check: is this code called in ways that constrain how it can change?

## Understanding Architecture or "How Does X Work"

1. Start from directory structure and entry points
2. Read types/interfaces to understand module contracts
3. Trace one concrete request or operation end to end
4. Note the architectural layers and how they communicate
5. Check: are there architectural docs, ADRs, or README files that explain intent?
