# Working Model Slices

A working model is a set of task-relevant relationships the agent can name before editing. It is not a repository summary. Build only the slices needed to choose the change point, shape the implementation, predict impact, and verify the result.

## Core Slices

| Slice | What to establish | Evidence examples | Enough when |
|---|---|---|---|
| Task boundary | Behavior to add, fix, remove, or preserve; explicit non-goals | user request, issue text, failing test, error output | in-scope and out-of-scope behavior can be named |
| Entry path | How the behavior is reached | route, command, handler, public API, job, test invocation | at least one concrete path reaches the relevant behavior |
| Behavior owner | Code that makes the decision or changes state | conditional, transformation, mutation, I/O, validation, contract enforcement | the candidate edit location is behavior-changing, not only a wrapper |
| Data and state | Relevant values, shapes, transformations, defaults, persistence, and branch controls | types, schemas, migrations, fixtures, serializers, config definitions | the source, shape, transformations, and controlling values are named |
| Consumers and contracts | Callers, users, implementations, persisted readers, or API consumers and their assumptions | references, implementations, tests, API schemas, docs | impacted consumers are named, or untraced consumers are explicitly bounded |
| Local conventions | How analogous code is organized, named, registered, tested, and generated | similar feature, registry, base class, helper, fixture, package config | the new or changed code has an existing pattern to follow, or no analogue was found |
| Verification | How the change can be proven | failing test, focused new test, existing suite, lint/build command, manual check | at least one verification path is named, or the gap is recorded |
| Change shape | Whether the edit is direct, preparatory, split, or behavior-preserving | current contract, caller count, complexity, test coverage | implementation can proceed without mixing unrelated refactor and behavior changes |
| Unknowns and risk | What remains untraced and why it is acceptable or blocking | unexplored dynamic path, missing test, external service, generated code | unknowns are either off the critical path or must be resolved before editing |

## Depth Rules

- Deepen a slice when changing it would change the edit location, implementation shape, consumer risk, or verification path.
- Do not deepen a slice only because it is interesting; curiosity without an implementation decision attached is repository touring.
- If a slice cannot be established from local evidence, mark it as unknown and name the next concrete lookup or verification gap.
- For explanation-only tasks, build the slices needed to explain the behavior and name untraced branches that could change the explanation; do not force implementation-only slices such as change shape when no edit is being made.
