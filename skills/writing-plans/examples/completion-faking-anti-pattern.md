# Anti-Pattern: Completion-Faking

A plan can look orderly while failing either approval or execution.

## Human-readability faking

```markdown
## Decision Brief

**Intent:** Add CSV import.
**Outcome:** CSV import is added.
**Scope:** CSV import behavior.
**Visible Changes:** The system supports CSV import.
**Approval Needed:** None.

## Design Review

Use a clean import architecture with clear responsibilities and robust validation.
```

**Why this fails**

- Intent, Outcome, Scope, and Visible Changes paraphrase one fact instead of answering different approval questions.
- `Approval Needed: None` creates a field with no reader action.
- Design Review has no observable gate trigger, proposed boundary, material alternative, or contract.
- “Clean,” “clear,” and “robust” cannot be verified and hide the actual behavior and ownership choices.
- The headings are complete, but a human still cannot identify import behavior, preserved behavior, or a decision to approve.

## Execution completion-faking

```markdown
## Task 1: Set up the importer

This is to set up the CSV importer.

Files: `imports/csv.py`

- [ ] Create the importer
- [ ] Write tests
- [ ] Run the tests

## Task 2: Configure it

This is to configure column mapping.

Files: `config.yaml`, `imports/csv.py`

- [ ] Add config support
- [ ] Test config behavior
```

**Why this fails**

- The purpose sentences restate the task names instead of explaining what behavior or risk each task addresses.
- "Run the tests" is not executable verification; a cold-start agent needs the exact command and expected output.
- "The importer" depends on planning-conversation memory. The task does not state the input contract, accepted output, first action, or test assertion.
- The structure is present, but the document does not transfer enough knowledge for execution.

Adding more sections does not fix either failure. Replace duplicated approval text with a concrete Intent, Outcome, and Scope; replace vague execution steps with exact actions and verification.
