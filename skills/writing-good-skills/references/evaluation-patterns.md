# Capability Skill Evaluation Patterns

Use this reference when skill behavior can be exercised, two versions need comparison, or a plausible draft must be tested for actual value.

## Evaluate Two Independent Surfaces

### Discovery

Test whether the skill loads for intended tasks and remains inactive for adjacent tasks. Use realistic should-trigger and near-miss should-not-trigger prompts. Measure false negatives and false positives separately because they require different description changes.

### Execution

Run the same task in clean contexts under:

1. no skill, when creating a new capability;
2. the previous skill version, when optimizing an existing capability;
3. the candidate skill.

Keep the model, tools, task inputs, and environment fixed. Repeat runs when model variance could change the conclusion.

## Build Discriminating Cases

Include:

- representative tasks from the claimed task family;
- unseen instances that share structure but not surface wording;
- edge cases tied to known failure modes;
- adjacent tasks where the skill should not interfere;
- compatibility cases for supported versions or environments.

Do not let evaluation prompts reveal the intended method or expected fix unless that information would exist in real use.

## Measure Benefit And Cost

Record:

- task completion and acceptance-criterion pass rate;
- output quality that matters to the user;
- trigger accuracy;
- scope expansion, unsafe action, or other negative transfer;
- tokens, elapsed time, and tool calls when material;
- variance across repeated runs.

Use scripts or executable tests for mechanical outcomes. Use blind comparison or human review for usefulness, design judgment, visual quality, and other properties that cannot be reduced to robust assertions. Require concrete evidence for both pass and failure decisions.

## Interpret The Delta

- If both baseline and candidate pass, remove or challenge the instruction claimed to add value.
- If only the candidate passes, trace which instruction, reference, or script caused the improvement.
- If results vary, inspect traces for ambiguous wording, too many options, or nondeterministic evaluation.
- If the candidate costs more without meaningful gain, simplify it.
- If it harms adjacent tasks or conflicts with current context, narrow, update, branch, or retire it.

Do not patch every failed example with another rule or add neighboring domain risks that the task contract and evaluations do not exercise. Generalize from the observed failure mechanism, make the smallest candidate change, then rerun the full suite to detect regressions.

Call a skill qualified only when it shows meaningful net benefit on the target distribution. When evaluation is not yet possible, label it unverified and identify the next test that could distinguish value from plausible prose.
