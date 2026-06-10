# Concrete Design Sketch Examples

Read this reference when a plan needs a Concrete Design Sketch and the expected granularity is unclear. The sketch should show architecture shape, not full implementation.

## Contents

- [Granularity Rule](#granularity-rule)
- [Backend Service Example](#backend-service-example)
- [Frontend Component And State Example](#frontend-component-and-state-example)
- [CLI / Pipeline Example](#cli--pipeline-example)
- [Too-Detailed Anti-Pattern](#too-detailed-anti-pattern)

## Granularity Rule

A good sketch lets a cold-start implementation agent answer these questions before editing:

- Which files or modules own the new behavior?
- What are the important interfaces, component boundaries, or data contracts?
- What call, event, or data flow should the implementation follow?
- Where must state, domain logic, persistence, side effects, and orchestration not drift?
- What tempting implementation shape would satisfy the visible behavior but violate the architecture?

Do not specify local algorithms, incidental variables, complete code bodies, or future abstractions that are not needed for the current plan.

Use exact new filenames, method names, and type names only when repository evidence or an already-decided architecture contract justifies them. Otherwise name the directory/module responsibility and leave exact names to execution-time judgment.

## Backend Service Example

Good:

```text
## Concrete Design Sketch

Target shape:
  src/runs/run-controller.ts
    HTTP/input boundary only. Parse request and serialize response.
  src/runs/run-service.ts
    Orchestrates project lookup, policy decision, persistence, and queueing.
  src/runs/run-policy.ts
    Pure domain decision: whether a run can start.
  src/runs/run-repository.ts
    Persistence adapter only.

Boundary surfaces:
  RunService.startRun(input: StartRunInput): Promise<StartRunResult>

  RunPolicy.canStart(project, input):
    -> { type: "allow"; normalizedInput }
    -> { type: "reject"; reason }

Intended flow:
  runController.start(req):
    input = parseStartRunRequest(req)
    result = runService.startRun(input)
    return toStartRunResponse(result)

  runService.startRun(input):
    project = projectRepository.get(input.projectId)
    decision = runPolicy.canStart(project, input)
    if decision.type == "reject":
      return rejectedRun(decision.reason)

    run = runRepository.create(decision.normalizedInput)
    queue.enqueue(run.id)
    return startedRun(run)

Ownership:
  - controller does not contain run eligibility logic
  - policy does not read or write persistence
  - repository does not know queue behavior
  - service is the only place that coordinates policy, persistence, and queueing

Shape to avoid:
  runController.start(req):
    read project
    check eligibility
    insert run
    enqueue job
    return response
```

Bad:

```text
## Concrete Design Sketch

- Keep the service architecture clean.
- Put business logic in the right layer.
- Avoid coupling controllers and persistence.
- Reuse existing abstractions where appropriate.
```

Why it fails: every sentence is plausible, but none identifies the file shape, boundary methods, call flow, or ownership split an implementation agent must follow.

## Frontend Component And State Example

Good:

```text
## Concrete Design Sketch

Component shape:
  SettingsPage
    owns loading/error/save lifecycle and calls settingsService
    renders SettingsToolbar, SettingsForm, AdvancedSettingsPanel

  SettingsForm
    owns local draft field values only
    emits onSubmit(draft)

  AdvancedSettingsPanel
    receives derived props and emits field changes
    does not fetch, save, or write global store state

Boundary surfaces:
  SettingsFormProps:
    initialValue: SettingsDraft
    disabled: boolean
    onSubmit(draft: SettingsDraft): void

  settingsService.updateSettings(draft): Promise<Settings>

Intended flow:
  SettingsForm.onSubmit(draft)
    -> SettingsPage.handleSave(draft)
    -> settingsService.updateSettings(draft)
    -> settingsStore.refresh()
    -> SettingsPage displays success/error state

Ownership:
  - SettingsPage owns async lifecycle and side effects
  - form components own only local draft interaction state
  - settingsService owns API calls
  - global store is refreshed after successful save, not mutated by the form

Shape to avoid:
  SettingsForm.onSubmit(draft):
    call API directly
    mutate settingsStore
    show toast
    navigate or update router state
```

Bad:

```text
## Concrete Design Sketch

- Use clean React components.
- Keep state close to where it is used.
- Avoid prop drilling.
- Make saving robust.
```

Why it fails: it sounds like design guidance, but it does not stop the implementation from putting API calls, global store writes, and toast behavior inside the form.

## CLI / Pipeline Example

Good:

```text
## Concrete Design Sketch

Target shape:
  cli/import-command.ts
    Parses flags and prints user-facing output.
  imports/import-service.ts
    Orchestrates parse, validation, persistence, and reporting.
  imports/import-validator.ts
    Pure validation of parsed records.
  imports/import-repository.ts
    Persistence only.

Boundary surfaces:
  ImportService.runImport(input: ImportInput): Promise<ImportReport>

  ImportReport:
    importedCount
    skippedCount
    validationErrors[]

Intended flow:
  importCommand.run(argv):
    input = parseImportFlags(argv)
    report = importService.runImport(input)
    return printImportReport(report)

  importService.runImport(input):
    records = importParser.parse(input.file)
    validation = importValidator.validate(records)
    if validation.hasFatalErrors:
      return failedReport(validation.errors)

    persisted = importRepository.upsert(validation.validRecords)
    return successReport(persisted, validation.warnings)

Ownership:
  - CLI command does not validate record semantics or write persistence
  - validator does not know CLI flags or persistence
  - repository does not format user-facing reports
  - service coordinates the pipeline

Shape to avoid:
  importCommand.run(argv):
    parse file
    validate records
    write database
    format report
```

Bad:

```text
## Concrete Design Sketch

- Add an import service.
- Keep CLI thin.
- Validate before writing.
```

Why it fails: "service" and "thin CLI" are labels, not an executable architecture shape. The plan still leaves validation, persistence, and reporting ownership ambiguous.

## Too-Detailed Anti-Pattern

Bad:

```text
## Concrete Design Sketch

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

Why it fails: it starts writing implementation code before execution has read the exact local constraints. Keep the sketch at module, boundary, ownership, and flow level unless a local detail is an approved contract.
