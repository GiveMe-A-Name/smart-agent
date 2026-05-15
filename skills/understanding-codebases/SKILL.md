---
name: understanding-codebases
description: "Build evidence-based understanding of code before suggesting changes. TRIGGER when: current understanding is based on descriptions, user explanations, or prior knowledge — not read source files."
---

# Codebase Understanding

Build grounded, task-relevant understanding of the code before suggesting implementation — focused on the slice that matters to the current question, not a tour of the whole repository.

## Trigger Logic

**Invocation default**: The cost of an unnecessary invocation is minor overhead. The cost of a missed invocation is unfounded conclusions that contaminate all subsequent work. When the task is not confirmed — from already-seen code evidence, not impression or prior knowledge — to involve only a single self-contained file with no cross-file tracing needed, invoke this skill.

**What counts as code evidence**: reading actual source files and tracing behavior. Issue descriptions, PR summaries, bug reports, and user explanations are hypotheses about where the problem is — not evidence of what the code does.

Use this skill when:
- answering safely requires tracing behavior across multiple files or layers
- the question is concrete, but you cannot yet point to source files or traced behavior that answer it
- proceeding would require guessing about code structure, ownership, or behavior

Do not use this skill when:
- you have already read the relevant source files and can state the answer from that evidence [re-running investigation does not improve accuracy — it delays output and inflates apparent thoroughness]
- the question is about intent, scope, or acceptance criteria rather than code behavior [those require clarification from the user, not code investigation]

**Before deepening exploration**, distinguish:
- `intent ambiguity`: the request mainly needs clarification — but if code evidence is thin, explore first, then clarify
- `repo-fact gap`: the question is concrete, but repository evidence is missing — use this skill
- `already answerable`: you have already run targeted lookups that produced concrete, sufficient code evidence — do not invoke

## Boundary

This skill owns:
- building a task-specific repository slice for the current question
- tracing enough call flow to ground ownership, behavior, or candidate change points
- discovering codebase conventions and patterns relevant to the task
- comparing nearby implementations when local patterns matter
- stating what is proven, hypothesized, and still unknown

This skill does not own:
- clarifying product intent, scope, or acceptance criteria
- writing a change plan or sequencing implementation work
- final engineering judgment about how a change should be implemented
- recommending verification strategy or completion judgment

## Invariants

- **Never treat issue descriptions, PR text, or user explanations as code evidence** — they are hypotheses about where the problem is, not proof of what the code does [because descriptions reflect the author's mental model of the problem, not what the code actually does — grounding work on them produces investigations that confirm the hypothesis rather than test it].
- **Every search or file read must test a stated hypothesis** — not satisfy curiosity or fill in general context [because undirected exploration produces information about the codebase, not answers to the question — it expands context without closing the evidence gap].
- **Never stop at a facade, wrapper, or registration layer** — trace until you reach code that makes decisions, transforms data, or mutates state [because facades only confirm that something exists; behavior-changing code is where ownership, correctness, and candidate change points actually live].
- **Stop as soon as all Completion Criteria are met** — continuing to explore after the question is grounded is undirected browsing, not investigation [because additional context past the grounding point inflates apparent thoroughness without improving the accuracy of conclusions].

## Decision Signals

**When to widen scope**: expand beyond the initial slice only when a current lookup reveals dependencies that block answering the question — not by default.

**When to stop exploring**: stop when the last two file reads or searches produced no new call edge, behavior-changing function, ownership clue, convention example, or candidate change point. Remaining unknowns will surface during implementation; that is normal.

**When to consult git history**: when two behavior paths appear to handle the same case, names do not distinguish responsibilities (e.g., `handle`, `process`, `doWork`), or a behavior-changing branch has no local comment, test, or caller explaining why it exists. `git blame` plus the commit message often explains what application code alone cannot.

**When to read infrastructure/config**: when deployment constraints, environment variables, or build system shape could affect the answer. For add-a-feature investigations, read the project's build configuration (the file that defines package structure, tooling, and linting — e.g. `pyproject.toml`, `Cargo.toml`, `package.json`) and test infrastructure (fixture files, test configuration) by default — "could this affect the answer?" resolves to yes for nearly all feature additions [because these files define package discovery, linting gates, and testing patterns that new code must comply with; reading only source files produces a technically correct but practically incomplete picture of what adding the feature correctly requires].

**When to check for runtime overrides**: when reading a value that determines behavior — a path, threshold, mode, or feature flag — check whether it can be overridden at runtime via environment variable, CLI argument, config file, or feature toggle. Document both the default AND the override mechanism; the override name is as important as the default value. A value that looks fixed in source code may be configurable in deployment — reporting only the hardcoded default produces a factually incomplete description.

**When the answer is a guide, not an audit**: if the question is "how do I add X", "how do I replicate X", or "what should I know before doing X" — each evidence item must state its practical implication, not just the mechanism. Not "discovery uses importlib" but "discovery uses importlib, therefore no registration step is needed." Stop before responding if any evidence item lacks a task-specific implication.

**How much P/H/U to write**: use one Unknown sentence when the traced path is single-entry, single-exit, and has no unresolved branch relevant to the question. Use fully expanded P/H/U when multiple paths, unresolved ownership, configuration overrides, or tradeoffs remain.

**When a hypothesis is not new**: a new hypothesis changes *what* you are looking for, not just *where*. If the second lookup tests the same absence in a different location, it is not a new hypothesis — stop and report the gap. See `examples/fake-new-hypothesis.md`.

## Completion Criteria

- [ ] Every conclusion names at least one source file, line, command result, or traced call path that supports it.
- [ ] Which files matter and the role each plays is stated — not just listed.
- [ ] The call flow relevant to the question is traced.
- [ ] The conventions the codebase uses for similar work are identified.
- [ ] Candidate change points are identified with evidence for why they are correct.
- [ ] What is still uncertain is stated explicitly. For broad questions (architecture, end-to-end flow, cross-module behavior), separate proven/hypothesized/unknown in full. For focused questions (single bug trace, single function behavior), one sentence naming what cannot be confirmed from code is sufficient.
- [ ] For large codebases, exploration was scoped to the task-relevant slice, not an exhaustive sweep.
- [ ] Infrastructure and config files were checked if deployment constraints or implicit dependencies are relevant. For add-a-feature questions, build configuration and test infrastructure were read — not just source files.
- [ ] **Actionability (action-oriented questions only)**: if the question is "add X", "replicate X", or "what should I know before doing X" — each evidence item names its practical implication for the reader's task. Stop if evidence items appear without a practical implication.

If any conclusion cannot be backed by a file, line, command result, or traced call path already read, keep investigating or mark it as unknown.

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is evidence-accumulation-type: completion is verified by self-checking accumulated evidence against the Completion Criteria checklist. The output is not a document but a grounded evidence base — completion means every checklist item can be confirmed by pointing to specific files, line numbers, or traced call paths already read, not by assertion.

## Anti-Rationalization Check

This section exists because coherent explanations feel like grounded understanding — but coherence is a property of the explanation, not the evidence. A plausible mental model and a code-backed conclusion are indistinguishable from the inside; only external evidence distinguishes them.

Pause before exiting.

Am I treating a plausible mental model as if it were code-backed evidence?

Am I exiting because understanding is genuinely grounded, or because the current explanation feels coherent enough?

Completion-faking signals specific to codebase investigation — stop if any apply:
- Conclusions reference file names or function names without having read those files in this session
- The call flow is described as "likely" or "probably" rather than traced to actual code
- Candidate change points were identified from the directory structure or naming, not from reading behavior-changing code
- "What is still unknown" was left empty — investigations always have unknowns; an empty section means they were not looked for

## Self-Correction Signals

Stop and revise when:

- you are suggesting changes before building a local repository slice
- you stopped at an export, registration, config, or facade layer without tracing to behavior-changing code [because these layers confirm existence, not behavior — stopping here produces a map of what exists, not an understanding of what it does]
- you listed files without explaining relationships or call flow [because a file list is an index, not evidence — understanding requires knowing how the files interact, not that they exist]
- you are treating guesses about ownership or change placement as facts
- you skipped pattern discovery — proposing an approach without checking how the codebase already handles similar cases
- you are doing undirected exploration without a hypothesis to test [because exploration without a hypothesis produces context about the codebase, not answers to the question — and context without answers is a stopping condition, not progress]
- you are drifting into planning or still exploring after the question is already grounded
- you are trying to understand the entire codebase before acting on a localized task
- you read a configurable value and reported only the default, without noting the override mechanism in the same assignment [because a value with a runtime override is not a fixed value — reporting only the default produces a description that is wrong for any non-default deployment]
- you are listing "all files that use X" when the question is "how is X set up" or "how do I replicate X" — before reading the next consumer file, ask: if this file confirms it uses X the same way I expect, will the user's question be better answered? If no — skip [because consumer enumeration answers "who depends on X", not "how X works"; conflating the two expands scope without closing the evidence gap]

---

For investigation technique reference, see `references/investigation-techniques.md`.
For task-type starting strategies, see `references/task-driven-investigation.md`.
For large codebases, infra code, and poorly-named code, see `references/challenging-situations.md`.
For concrete investigation examples, see `examples/`.
