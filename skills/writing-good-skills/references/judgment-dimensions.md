# Capability Skill Judgment Dimensions

Use this reference when the capability gap, task-family boundary, control level, or compatibility risk is unclear. Do not open it for a straightforward wording edit with an already-established behavioral target.

## 1. Capability Gap

Ask what changes when the skill is absent.

Strong evidence includes repeated agent corrections, failed task traces, recurring reinvention of the same helper, domain facts unavailable to the model, project conventions that contradict generic defaults, or a successful expert workflow the agent cannot reproduce reliably.

Do not create a skill merely because a topic is important. If baseline runs already satisfy the target reliably, extra instructions add context cost without adding capability. Treat plausible concerns outside the observed gap and explicit task contract as a backlog, not as justification for a comprehensive first draft.

## 2. Coherent Task Family

Group tasks when they share the same specialized knowledge, decision structure, tools, and success criteria. A database query skill may include schema navigation and result formatting when those are always part of the same task. Database administration belongs elsewhere if it has different permissions, risks, and success conditions.

Reject both extremes:

- Instance-specific material encodes an answer instead of a reusable method.
- Topic-wide material loads unrelated rules and creates false triggers or conflicting instructions.

Use unseen examples as the transfer test: the skill should guide a structurally similar case without containing its answer.

## 3. Control Calibration

Choose control from task evidence, not a preferred document structure.

- Use principles and tradeoffs when context determines the right path.
- Use a default plus escape condition when one choice is usually best.
- Use checklists when omissions are the main failure mode.
- Use exact procedures when order is part of correctness.
- Use tested scripts when mechanical logic is repeated or deterministic reliability matters.

A capability skill may mix all five. Over-control causes wasted work and poor adaptation; under-control causes inconsistency and skipped safety or correctness gates.

## 4. Knowledge Placement

Keep content in the main file when every valid invocation needs it or the agent would not recognize when to seek it. Move variant-specific procedures, long examples, schemas, and lookup tables to focused references with observable read conditions. Bundle repeated computation or validation as scripts rather than asking the agent to recreate it.

Each important idea needs one authoritative home. Repetition increases attention competition and makes later edits inconsistent.

## 5. Compatibility And Negative Transfer

Identify assumptions about model, client, tool availability, operating system, repository conventions, dependency versions, permissions, and data shape. Encode a compatibility condition when the skill would otherwise apply incorrect guidance.

Test adjacent tasks and environments, not only happy paths. A skill that improves its primary example but degrades neighboring tasks, expands scope, or conflicts with current project evidence has negative transfer. Narrow, branch, update, or retire it instead of adding defensive prose around an incompatible core.
