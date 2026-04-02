---
name: understanding-codebases
description: "Invoke when the task involves working on code and current understanding is based on issue descriptions, user explanations, or prior knowledge — not read source files. Cost of unnecessary invocation: minor overhead from a targeted exploration. Cost of missing: unfounded conclusions that contaminate all subsequent planning, investigation, and implementation. When uncertain whether evidence is sufficient, invoke."
---

# Codebase Understanding

Build grounded, task-relevant understanding of the code before suggesting implementation — focused on the slice that matters to the current question, not a tour of the whole repository.

This skill teaches how to investigate code with purpose: identify the files and modules that matter, trace how behavior flows, discover the patterns and conventions the codebase follows, and explain where a change likely belongs. The result is a grounded understanding with clear separation between what is proven, what is hypothesized, and what is still unknown.

## Trigger Logic

**Invocation default**: The cost of an unnecessary invocation is minor overhead. The cost of a missed invocation is unfounded conclusions that contaminate all subsequent work. When the task is not confirmed — from already-seen code evidence, not impression or prior knowledge — to involve only a single self-contained file with no cross-file tracing needed, invoke this skill.

**What counts as code evidence**: reading actual source files and tracing behavior. Issue descriptions, PR summaries, bug reports, and user explanations are hypotheses about where the problem is — not evidence of what the code does. A hypothesis that "the problem is in module X" does not confirm scope; it names where to look first.

Use this skill when:
- answering safely requires tracing behavior across multiple files or layers
- the question is concrete, but current repository evidence is too thin to answer confidently
- proceeding would require guessing about code structure, ownership, or behavior

Before deepening exploration, distinguish among these cases:

- `intent ambiguity`: the request mainly needs clarification or human decisions; repository exploration does not replace that, but it often informs what questions to ask — if codebase evidence is thin, explore first, then clarify
- `repo-fact gap`: the question is concrete, but repository evidence is missing; use this skill
- `already answerable`: you have already run targeted lookups that produced concrete, sufficient code evidence; do not invoke this skill

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

- Do not proceed based on assumptions; ground decisions in code evidence
- Issue descriptions, PR text, and user explanations are hypotheses, not evidence
- Every search or file read must test a stated hypothesis — not satisfy curiosity
- Stop once the answer is grounded well enough to avoid intuition

## Completion Checklist

- [ ] Did I ground conclusions in code evidence, not assumptions or prior knowledge?
- [ ] Did I separate what is proven, hypothesized, and still unknown?
- [ ] Did I check how the codebase handles similar cases (pattern discovery)?
- [ ] For large codebases: did I scope exploration to the task-relevant slice, not attempt exhaustive understanding?
- [ ] Did I check infrastructure/config files when deployment constraints or implicit dependencies are relevant?
- [ ] Am I exiting because understanding is genuinely grounded, or rationalizing?

**If any check fails, return to the relevant section before exiting.**

## Investigation Techniques

These are techniques to select from based on the task — not a fixed sequence. Choose the ones that answer the current question most efficiently.

### Anchor on Entry Points

Start from where behavior begins. Entry points vary by codebase type:
- **Web apps**: route handlers, API endpoints, middleware chains
- **CLI tools**: command definitions, argument parsers
- **Libraries**: exported public API, main module index
- **Event-driven**: event listeners, message handlers, webhook receivers

From the entry point, trace inward toward where decisions are made and state changes.

### Trace the Delegation Chain

Most entry points delegate to deeper layers. Trace through the layers until you reach code that makes decisions or changes state — not just code that routes or configures.

Recognize and move past these delegation layers quickly:
- **Facades/wrappers**: re-export, rename, or add minimal logic
- **Configuration layers**: wire dependencies together but contain no business logic
- **Middleware/interceptors**: modify context but rarely own core behavior
- **Adapters**: translate between interfaces without adding decisions

Stop when you reach code that contains conditional logic, data transformations, or state mutations relevant to your question.

### Analyze Consumers (Reverse Tracing)

When you need to understand a piece of code's impact or role, trace who calls it:
- Use "Find References" / search for function/class name across the codebase
- Distinguish direct callers from transitive dependents
- Note whether the code is called from one place (likely safe to change) or many (change requires broader understanding)

This is essential when evaluating whether a candidate change point is the right one — a function called from 15 places carries different risk than one called from 2.

### Discover Patterns and Conventions

Before proposing how to implement something, find how the codebase already does similar things:
- Search for analogous features (e.g., if adding a new API endpoint, find existing endpoints)
- Note naming conventions, file organization patterns, error handling approaches
- Look at how tests are structured for similar functionality
- Check for shared utilities, base classes, or helper functions that your code should use

Following existing patterns is usually the right default. Deviating should be a conscious, justified decision — not ignorance.

### Read Types, Interfaces, and Schemas

Type definitions, interfaces, and schemas are the fastest way to understand a module's contract without reading every line of implementation:
- **Typed languages**: interface files, type definitions, abstract classes
- **API boundaries**: OpenAPI specs, GraphQL schemas, protobuf definitions
- **Database**: migration files, model definitions, schema files
- **Config**: config types, environment variable definitions

These tell you what data flows where and what each module promises to its consumers.

### Use Tests as Documentation

Test files often reveal intended behavior more reliably than comments or docs:
- Test names describe expected behavior in concrete terms
- Test fixtures show realistic input/output examples
- Edge case tests reveal boundary conditions the original author considered
- Missing tests reveal areas of uncertainty or risk

When a module's purpose is unclear, its tests are often the best explanation.

### Consult History When "Why" Matters

When code seems strange, over-engineered, or contradictory, the answer is often in its history:
- **git blame**: who wrote this line and when — follow to the commit message
- **Commit messages and PR descriptions**: explain the intent behind a change
- **Recent change frequency**: files changed often may be unstable or central
- **Large refactoring commits**: reveal architectural decisions and migration patterns

Use history to understand design decisions — not to audit code line by line.

### Read Directory Structure for Architecture

Before diving into files, spend a moment understanding how the project is organized:
- Top-level directories often map to architectural layers or domains
- `package.json`, `Cargo.toml`, `go.mod`, or equivalent reveal dependencies and project shape
- Monorepo structures reveal module boundaries
- Shared/common directories reveal cross-cutting utilities

This gives you a mental map that makes subsequent file-level investigation faster.

## Task-Driven Investigation

Different tasks have different optimal starting points. Choose the strategy that matches the task:

**Fixing a bug:**
1. Start from the symptom (error message, wrong behavior, failing test)
2. Search for the symptom in code (error strings, variable names, log messages)
3. Trace the execution path that produces the symptom
4. Identify where the actual behavior diverges from expected behavior
5. Check: are there tests that should have caught this? What do they test?

**Adding a feature:**
1. Find how similar features are implemented (pattern discovery)
2. Identify the conventions: file location, naming, registration, testing
3. Trace the data flow for a similar feature end to end
4. Identify the extension points where your feature plugs in
5. Check: what shared utilities or base classes should you use?

**Understanding for refactoring:**
1. Map the dependency graph of the code in question (who calls what)
2. Identify all consumers of the code you plan to change
3. Understand the implicit contracts (what callers assume)
4. Look for tests that verify the current behavior
5. Check: is this code called in ways that constrain how it can change?

**Understanding architecture or "how does X work":**
1. Start from directory structure and entry points
2. Read types/interfaces to understand module contracts
3. Trace one concrete request or operation end to end
4. Note the architectural layers and how they communicate
5. Check: are there architectural docs, ADRs, or README files that explain intent?

## Challenging Codebase Situations

The investigation techniques and task-driven strategies above work well for reasonably structured codebases. These situations require additional judgment.

### Large Codebases — Knowing When "Enough" Is Enough

In a large codebase (hundreds of thousands or millions of lines), you cannot understand everything before acting. The skill is progressive comprehension — understanding just enough for the current task, then deepening as needed.

**Key signals**:
- **Start narrow, widen on evidence.** Begin with the smallest slice that could answer your question. Only expand scope when the current slice reveals dependencies you cannot ignore. "Let me understand the whole system first" is not a strategy — it's procrastination.
- **Use module boundaries as natural stopping points.** If your task is within one module, you need to understand that module's contract (interfaces, types, public API) and its immediate dependencies — not the entire dependency tree. Trust the abstraction boundary until evidence suggests it's leaky.
- **Recognize the diminishing returns curve.** The first 20% of investigation gives you 80% of the understanding. When each new file you read adds only marginal information, you likely have enough to proceed. The remaining unknowns will surface during implementation — that's normal, not a sign of insufficient research.
- **Track what you've explored and what you haven't.** In a large codebase, it's easy to lose track of what you've already investigated. Maintain a mental (or written) map of: explored and understood, explored but uncertain, known to exist but not explored, unknown.

### Infrastructure and Configuration Code

Dockerfiles, CI/CD pipelines, Infrastructure-as-Code (Terraform, CloudFormation), and build configurations are code too — but they follow different conventions than application code.

**Key signals**:
- **Read infrastructure code for intent, not syntax.** A Dockerfile tells you how the application is built and deployed. A CI config tells you what quality gates exist. A Terraform file tells you what cloud resources the system depends on. These are architecture documents disguised as code.
- **Trace the deployment pipeline.** Understanding how code gets from a commit to production often reveals constraints that application code alone doesn't show: which tests run, what environments exist, what approvals are needed, what resources are provisioned.
- **Configuration files reveal implicit dependencies.** Environment variables, secrets management, service discovery configs, and feature flags all indicate external dependencies and runtime behavior that doesn't appear in the application source.
- **Build system configuration reveals project structure.** Monorepo configs (workspace definitions, build targets), bundler configs, and compiler settings reveal module boundaries, dependency relationships, and build order — often more reliably than documentation.

### Poorly Documented or Poorly Named Codebases

When variable names are cryptic, comments are absent or misleading, and no documentation exists, standard investigation techniques still work — but require more inferential reasoning.

**Key signals**:
- **Tests are the most reliable documentation.** When code is poorly documented, test files reveal intended behavior through concrete examples. Test names (even bad ones) hint at what the author thought was important. Test fixtures show realistic data shapes.
- **Git history compensates for missing comments.** When a function is named `processData` and does something non-obvious, `git blame` and the corresponding commit message or PR description often explain what the original author intended. Large refactoring commits reveal architectural intent.
- **Trace data flow when names are unhelpful.** When `x` is passed to `handleThing` which calls `doStuff`, trace the actual data: where does `x` come from (API input? database record? config?), what type is it, what transformations does it undergo, where does the result go? Data flow reveals purpose even when names don't.
- **Look for the domain model.** Even in poorly named code, there is usually a core set of entities and their relationships. Find the database schema, the protobuf definitions, or the type declarations — these reveal the domain model that the application code operates on. Understanding the domain model makes the code comprehensible even when individual names are opaque.
- **Do not rename or refactor to understand.** It's tempting to rename variables while exploring to make code clearer. Resist this — renaming before understanding risks encoding wrong assumptions. Understand first, rename later (if the task calls for it).

## Grounded Understanding

You have enough understanding for the current question when, from code evidence, you can do the relevant subset of:

- explain which files matter and what role each plays (not just list them)
- trace the call flow relevant to the question
- point to the conventions the codebase uses for similar work
- identify the candidate change points with evidence for why they are correct
- state what remains unknown or unverified

If you still need intuition to bridge the answer, keep investigating.

## Self-Correction Signals

Stop and revise when:

- you are suggesting changes before building a local repository slice
- you stopped at export, registration, config, or facade layers without tracing deeper
- you listed files without explaining relationships or call flow
- you are treating guesses about ownership or change placement as facts
- you skipped pattern discovery — proposing an approach without checking how the codebase already handles similar cases
- you read implementation details but skipped types/interfaces that would have given you the contract faster
- you are doing undirected exploration without a hypothesis to test
- you are drifting into planning or still exploring after the question is already grounded
- you anchored your slice on a search result without verifying the exploration target is correct
- you are trying to understand the entire codebase before acting on a localized task — progressive comprehension, not exhaustive comprehension
- you skipped infrastructure/config files that reveal deployment constraints or implicit dependencies relevant to the task
- you are renaming or refactoring code as a way to understand it, before understanding is grounded

## Judgment in Practice

### Example 1: Tracing Behavior

**Task**: "Where does the auth token get validated?"

**Investigation strategy**: This requires entry point identification + delegation chain tracing.

1. **Hypothesis**: Auth validation likely starts in request middleware. Search for auth-related middleware.
2. **First finding**: `middleware/auth.ts` has a `validateToken` call that delegates to `services/token.ts`. This grounds the entry path, but the file delegates — not yet the behavior-changing code.
3. **Follow the delegation**: Read `services/token.ts` — it performs signature checks and expiry validation. This is the behavior-changing code.
4. **Result**: `middleware/auth.ts` is the entry point and orchestration layer; `services/token.ts` contains the actual validation logic. Any deeper delegation not directly confirmed stays labeled as hypothesis.

**What this prevents**: Stopping at the first plausible file; treating a function name like `validateToken` as proof of where validation really happens.

### Example 2: Adding a Feature by Pattern Discovery

**Task**: "Add a new webhook event type for user deletion"

**Investigation strategy**: This requires pattern discovery + convention tracing.

1. **Find a similar feature**: Search for an existing webhook event type (e.g., `user.created`). Find it defined in `events/user-created.ts`.
2. **Discover the pattern**: The event type is registered in `events/index.ts`, has a corresponding schema in `schemas/events/`, a handler in `handlers/webhooks/`, and a test in `tests/events/`.
3. **Trace the convention**: Events follow a naming pattern, use a base class `WebhookEvent`, register through a central registry, and have integration tests that verify delivery.
4. **Result**: To add `user.deleted`, follow the same pattern: create event definition, add schema, register it, add handler, write tests matching existing test structure. The extension point is the event registry, not the webhook dispatcher.

**What this prevents**: Implementing the feature in an ad-hoc way that ignores existing conventions; missing the registration step; putting code in the wrong layer.

See `examples/fake-new-hypothesis.md` for a contrast pair showing how a lookup that looks like a new hypothesis can actually just be rechecking the same absence. See `examples/undirected-vs-task-driven.md` for the difference between curiosity-driven browsing and hypothesis-driven investigation.

