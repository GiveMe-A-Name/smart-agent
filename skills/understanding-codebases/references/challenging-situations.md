# Challenging Codebase Situations

The investigation techniques and task-driven strategies work well for reasonably structured codebases. These situations require additional judgment.

## Large Codebases — Knowing When "Enough" Is Enough

In a large codebase (hundreds of thousands or millions of lines), you cannot understand everything before acting. The skill is progressive comprehension — understanding just enough for the current task, then deepening as needed.

**Key signals**:
- **Start narrow, widen on evidence.** Begin with the smallest slice that could answer your question. Only expand scope when the current slice reveals dependencies you cannot ignore. "Let me understand the whole system first" is not a strategy — it's procrastination.
- **Use module boundaries as natural stopping points.** If your task is within one module, you need to understand that module's contract (interfaces, types, public API) and its immediate dependencies — not the entire dependency tree. Trust the abstraction boundary until evidence suggests it's leaky.
- **Recognize the diminishing returns curve.** The first 20% of investigation gives you 80% of the understanding. When each new file you read adds only marginal information, you likely have enough to proceed. The remaining unknowns will surface during implementation — that's normal, not a sign of insufficient research.
- **Track what you've explored and what you haven't.** In a large codebase, it's easy to lose track of what you've already investigated. Maintain a mental (or written) map of: explored and understood, explored but uncertain, known to exist but not explored, unknown.

## Infrastructure and Configuration Code

Dockerfiles, CI/CD pipelines, Infrastructure-as-Code (Terraform, CloudFormation), and build configurations are code too — but they follow different conventions than application code.

**Key signals**:
- **Read infrastructure code for intent, not syntax.** A Dockerfile tells you how the application is built and deployed. A CI config tells you what quality gates exist. A Terraform file tells you what cloud resources the system depends on. These are architecture documents disguised as code.
- **Trace the deployment pipeline.** Understanding how code gets from a commit to production often reveals constraints that application code alone doesn't show: which tests run, what environments exist, what approvals are needed, what resources are provisioned.
- **Configuration files reveal implicit dependencies.** Environment variables, secrets management, service discovery configs, and feature flags all indicate external dependencies and runtime behavior that doesn't appear in the application source.
- **Build system configuration reveals project structure.** Monorepo configs (workspace definitions, build targets), bundler configs, and compiler settings reveal module boundaries, dependency relationships, and build order — often more reliably than documentation.

## Poorly Documented or Poorly Named Codebases

When variable names are cryptic, comments are absent or misleading, and no documentation exists, standard investigation techniques still work — but require more inferential reasoning.

**Key signals**:
- **Tests are the most reliable documentation.** When code is poorly documented, test files reveal intended behavior through concrete examples. Test names (even bad ones) hint at what the author thought was important. Test fixtures show realistic data shapes.
- **Git history compensates for missing comments.** When a function is named `processData` and does something non-obvious, `git blame` and the corresponding commit message or PR description often explain what the original author intended. Large refactoring commits reveal architectural intent.
- **Trace data flow when names are unhelpful.** When `x` is passed to `handleThing` which calls `doStuff`, trace the actual data: where does `x` come from (API input? database record? config?), what type is it, what transformations does it undergo, where does the result go? Data flow reveals purpose even when names don't.
- **Look for the domain model.** Even in poorly named code, there is usually a core set of entities and their relationships. Find the database schema, the protobuf definitions, or the type declarations — these reveal the domain model that the application code operates on. Understanding the domain model makes the code comprehensible even when individual names are opaque.
- **Do not rename or refactor to understand.** It's tempting to rename variables while exploring to make code clearer. Resist this — renaming before understanding risks encoding wrong assumptions. Understand first, rename later (if the task calls for it).
