# Detailed Design Examples

Read this reference when a plan needs Detailed Design and the expected granularity is unclear. The design should help a plan reader understand the chosen solution, abstraction model, flow, contract/data shape, and approval-critical logic without becoming an execution checklist.

## Contents

- [Granularity Rule](#granularity-rule)
- [Backend Service Example](#backend-service-example)
- [Frontend Component And State Example](#frontend-component-and-state-example)
- [CLI / Pipeline Example](#cli--pipeline-example)
- [Too-Detailed Anti-Pattern](#too-detailed-anti-pattern)

## Granularity Rule

A good Detailed Design lets a plan reader answer these questions before reading file maps or task checklists:

- What is this solution doing structurally?
- Why is this shape the chosen design?
- What will the post-change code architecture look like?
- Which responsibility blocks will exist after the change?
- What user action, event, data, or control flow connects those blocks?
- What contract/data shape is approval-relevant?
- What key branching, retry, fallback, validation, migration, state transition, or orchestration logic matters enough to show as pseudocode?
- Which boundaries are changed or preserved?
- What tempting shape would satisfy visible behavior but indicate the design was misunderstood?

Do not specify incidental variables, complete code bodies, task steps, line-level edits, or future abstractions that are not needed for understanding the approved design. Pseudocode is appropriate when it explains approval-critical logic; it is too detailed when it becomes production code or an edit script.

Use responsibility or module-area names. Avoid file paths, method names, and private type names unless they are public/developer-facing contract names being approved.

## Backend Service Example

Good:

```text
### Detailed Design

Design intent:
  Keep the HTTP boundary thin and keep run-start eligibility as a domain decision
  so future API formats do not duplicate persistence or policy logic.

Architecture diagram:
  API boundary -> run orchestration -> run policy
                                 -> persistence port -> persistence adapter
                                 -> queue port -> queue adapter

Target code architecture:
  API boundary package
    request parser / response serializer
    -> depends on run orchestration only

  Run domain package
    run orchestration
    run policy
    -> depends on persistence and queue ports

  Infrastructure adapters
    run persistence
    queue adapter

  Dependency direction:
    API boundary -> run orchestration -> policy / persistence port / queue port
    Infrastructure adapters do not import API request shapes.

Resulting responsibility shape:
  - Request boundary: parses the request and serializes the response.
  - Run orchestration: coordinates project lookup, policy decision, persistence,
    and queueing.
  - Run policy: decides whether a run may start; it does not persist data.
  - Run persistence: stores runs; it does not know queue behavior.

Main flow:
  API request -> request boundary -> run orchestration -> policy decision
  -> persistence -> queue -> API response

Contract/data shape:
  Start run request: project id, optional run name, requested queue.
  Start run result: accepted run id, rejected with policy reason, or missing project.

Key logic pseudocode:
  receive start request
  load project
  ask run policy whether the run may start
  if policy rejects:
    return rejected response without creating a run
  create run record
  enqueue run work
  return accepted run id

Boundaries changed or preserved:
  - eligibility logic belongs to the run policy responsibility, not the request boundary
  - persistence stays behind the run persistence responsibility
  - queueing is coordinated by orchestration, not by the persistence adapter

Misalignment shape:
  The request boundary reads the project, checks eligibility, inserts the run,
  enqueues work, and returns the response by itself.
```

Bad:

```text
### Detailed Design

- Keep the service architecture clean.
- Put business logic in the right layer.
- Avoid coupling controllers and persistence.
- Reuse existing abstractions where appropriate.
```

Why it fails: every sentence is plausible, but none lets the reader understand the resulting responsibility shape, flow, or ownership split.

## Frontend Component And State Example

Good:

```text
### Detailed Design

Design intent:
  Keep form components reusable and predictable by keeping save lifecycle and
  side effects at the page level.

Architecture diagram:
  Settings page -> settings service/store
                -> settings form components -> advanced settings panel

Target code architecture:
  Settings route/page
    owns loading, error, save lifecycle
    -> depends on settings service and renders form components

  Settings form components
    local draft interaction only
    -> emit submitted drafts and field changes

  Settings service/store
    service owns API calls
    store refreshes after successful save

  Dependency direction:
    page -> service/store and page -> form components
    form components do not depend on service/store/router.

Resulting responsibility shape:
  - Settings page: owns loading, error, save lifecycle, and calls the settings service.
  - Settings form: owns local draft field values only and emits submitted drafts.
  - Advanced settings panel: receives derived values and emits field changes.
  - Settings service: owns the API call.

Main flow:
  form submit -> page save handler -> settings service -> store refresh
  -> page shows success or error state

Contract/data shape:
  Settings draft: edited field values plus dirty state.
  Save result: success with refreshed settings, validation failure with field
  errors, or transport failure with page-level error feedback.

Key logic pseudocode:
  user edits fields in form draft
  when user submits:
    page marks save in progress
    call settings service with submitted draft
    if validation fails:
      show field errors and keep draft editable
    else if save fails:
      show page-level error and keep draft editable
    else:
      refresh store from saved settings
      show success state

Boundaries changed or preserved:
  - form components do not fetch, save, mutate global state, navigate, or show global feedback
  - the page owns async lifecycle and side effects
  - the global store is refreshed after successful save instead of being mutated by the form

Misalignment shape:
  The form submits by calling the API directly, mutating the global store,
  showing feedback, and navigating.
```

Bad:

```text
### Detailed Design

- Use clean React components.
- Keep state close to where it is used.
- Avoid prop drilling.
- Make saving robust.
```

Why it fails: it sounds like design guidance, but it does not let the reader see where save lifecycle, global state, and side effects will live.

## CLI / Pipeline Example

Good:

```text
### Detailed Design

Design intent:
  Keep CLI behavior focused on user interaction while import semantics remain
  reusable by other entry points.

Architecture diagram:
  CLI command -> import orchestration -> import validation
                                      -> persistence port -> persistence adapter

Target code architecture:
  CLI package
    import command
    -> depends on import orchestration only

  Import domain package
    import orchestration
    import validation
    -> depends on persistence port

  Infrastructure adapters
    import persistence

  Dependency direction:
    CLI -> import orchestration -> validation / persistence port
    validator and repository do not import CLI flag or output formatting logic.

Resulting responsibility shape:
  - CLI command: parses flags and prints user-facing output.
  - Import orchestration: coordinates parsing, validation, persistence, and reporting.
  - Import validation: validates parsed records without knowing CLI flags or persistence.
  - Import persistence: writes accepted records without formatting user-facing output.

Main flow:
  CLI flags -> import orchestration -> parse file -> validate records
  -> persist accepted records -> return import report -> CLI output

Contract/data shape:
  Import input: source path plus user-facing mode flags.
  Import report: accepted count, rejected count, validation messages, and
  persistence failure status.

Key logic pseudocode:
  CLI parses flags and source path
  pass import input to orchestration
  orchestration parses source records
  validate each record
  if validation errors exist and mode forbids partial import:
    return report without persisting
  persist accepted records
  return import report for CLI formatting

Boundaries changed or preserved:
  - CLI does not validate record semantics or write persistence
  - validator does not know CLI flags or persistence
  - repository does not format user-facing reports
  - import orchestration coordinates the pipeline

Misalignment shape:
  The CLI command parses the file, validates records, writes the database,
  and formats the report in one place.
```

Bad:

```text
### Detailed Design

- Add an import service.
- Keep CLI thin.
- Validate before writing.
```

Why it fails: "service" and "thin CLI" are labels, not a readable solution shape. The plan still leaves validation, persistence, and reporting ownership ambiguous.

## Too-Detailed Anti-Pattern

Bad:

```text
### Detailed Design

runService.startRun(input):
  const startedAt = Date.now()
  const project = await projectRepository.get(input.projectId)
  const normalizedName = input.name.trim().toLowerCase()
  const maxRetries = input.maxRetries ?? 3
  const metadata = {
    source: "api",
    createdBy: input.userId,
    timestamp: startedAt,
  }
  ...
```

Why it fails: it writes production-shaped implementation code before execution has read the exact local constraints. Keep Detailed Design at architecture, abstraction, flow, contract/data shape, and approval-critical pseudocode level unless a local detail is part of a public/developer-facing contract being approved.
