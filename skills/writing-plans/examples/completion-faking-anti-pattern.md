# Anti-Pattern: Completion-Faking

A plan can look orderly while failing as an execution artifact.

```markdown
## Task 1: Set up the notifier

This is to set up the notifier component.

Files: `notifications/webhook.py`

- [ ] Create the notifier
- [ ] Write tests
- [ ] Run the tests

## Task 2: Configure it

This is to configure the webhook.

Files: `config.yaml`, `notifications/webhook.py`

- [ ] Add config support
- [ ] Test config behavior
```

**Why this fails**
- The purpose sentences restate the task names instead of explaining what behavior or risk each task addresses.
- "Run the tests" is not executable verification; a cold-start agent needs the exact command and expected output.
- "The notifier" depends on planning-conversation memory. The task does not state the interface, payload, first action, or test assertion.
- The structure is present, but the document does not transfer enough knowledge for execution.
