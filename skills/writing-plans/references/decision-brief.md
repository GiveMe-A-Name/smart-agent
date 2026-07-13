# Decision Brief Reference

Read this reference when the required Decision Brief fields are hard to distinguish, a visible delta needs compression, or the draft repeats approval information. `SKILL.md` owns field definitions and triggers.

## Separate Intent, Outcome, And Scope

Use this test when the three required fields collapse into paraphrases:

- **Intent:** why the direction matters or which problem it addresses
- **Outcome:** the observable truth after execution
- **Scope:** where that truth applies and which plausible adjacent behavior stays unchanged

Bad:

```markdown
**Intent:** Add CSV export.
**Outcome:** CSV export is added.
**Scope:** CSV export behavior.
```

Better:

```markdown
**Intent:** Let operators analyze filtered reports without manually converting JSON.
**Outcome:** Users can download the current filtered report as CSV.
**Scope:** Report export formats only; querying, filters, and JSON output remain unchanged.
```

## Choose The Smallest Visible-Delta Form

Select the form that exposes the approval question with the least repetition:

- **Scenario bullets:** runtime conditions produce different outcomes.
- **Before/After:** one behavior changes directly.
- **Contract diff:** API, schema, event, CLI, or SDK consumers need exact compatibility information.
- **Behavior matrix:** multiple inputs combine into several outcomes.
- **Invariants:** preservation matters more than the new path.

Example—CLI behavior matrix:

| Input | Existing config | Result |
| --- | --- | --- |
| `--format json` | any | existing JSON output |
| `--format csv` | valid report | CSV download |
| unsupported format | any | parameter error |

Do not explain implementation or task order beneath the matrix; those belong in the Execution Plan.

## Compress Approval Choices And Risks

Approval Needed states the proposed choice and boundary in one sentence:

```markdown
**Approval Needed:** Apply the new permission only to administrative exports; ordinary report viewing remains unchanged.
```

Primary Risk names the observable uncertainty and how it can change the approved direction:

```markdown
**Primary Risk:** Existing integrations may parse unknown CLI fields strictly; verify compatibility before making the field default-visible.
```

If the choice or uncertainty does not affect outcome, scope, or design, keep it in execution risks rather than the Decision Brief.

## Duplication Check

Before finishing the brief:

- remove task titles, file paths, commands, and sequence
- merge facts repeated across Intent, Outcome, and Scope
- remove conditional fields with no reader action
- keep one compact representation of each visible delta
- delete any sentence whose removal does not change the approval decision

For a complete minimal bug-fix brief, see `examples/tiny-bug-fix.md`. For a contract-heavy brief, see `examples/medium-api-contract-change.md`.
