# Independent Plan Review Guidance

Use this guidance when a plan requires independent review. The reviewer checks whether the plan can survive execution by a fresh agent, not whether the document looks complete.

## Review Contract

The reviewer must read the plan as an execution artifact with two audiences:
- the human-facing section must let the user approve intent, scope, and strategy without codebase knowledge
- the execution section must let an agent start work cold without relying on the planning conversation

The review is not a rewrite request. It identifies blocking gaps, explains why each gap can break execution, and gives the smallest concrete correction needed.

## Required Context

Before reviewing, collect observable evidence:
- read the plan document from top to bottom
- if invoked as a sub-agent, read the review packet and use only its stated user intent, constraints, plan path, docs, and files as review scope
- read any architecture docs, specs, or repository instructions explicitly named by the plan
- read code files that the plan names as current-state evidence or modification targets
- if a referenced file does not exist, record that as an issue rather than inferring the intended file

A review without this context cannot catch repo-alignment failures [because the plan may be internally coherent while naming files, patterns, or constraints that do not exist].

Sub-agent context is intentionally incomplete. Do not infer unstated user goals, hidden conversation history, or preferred implementation details. Treat missing context as a finding only when the plan itself requires that context to execute, approve, or verify safely. Do not report a preference as an issue unless it identifies a specific execution failure mechanism tied to plan text, packet constraints, repository evidence, or a missing referenced artifact.

## What to Check

### 1. Human approval surface

Check whether the Human Review Section can be approved without codebase knowledge.

Block if:
- it omits the Intent statement
- Layer 1 omits End state, or End state is more than one sentence, describes implementation steps instead of the final observable state, or lists multiple concrete deltas that should be in Change Snapshot
- it contains file paths, line numbers, implementation-only function/method/class/module names, internal APIs, or non-user-visible CLI flags; public/developer-facing endpoints, fields, schema names, config keys, event names, CLI options, public SDK/plugin methods, and contract names are allowed only when they are the behavior being approved
- it describes implementation mechanics without naming the user problem, system behavior, or product/domain concept
- Change Snapshot appears in a tiny/small plan
- Change Snapshot is missing when a medium/large plan's intent, assessment, risks, Key Decisions, stop signals, or task outcomes include public/developer-facing contract changes, compatibility expectations, data/contract migrations, restored old behavior, multiple input/output outcomes, or multiple valid interpretations
- Change Snapshot omits approval-critical visible contract detail, includes implementation detail that belongs in the technical plan, or contains file paths, task references, implementation-only names, or explanatory prose; public/developer-facing endpoints, fields, schema names, config keys, event names, CLI options, public SDK/plugin methods, and contract names are allowed only when they are the behavior being approved
- medium/large plans omit Task Overview, Key Decisions or `None requiring human ratification`, or Conflict Priority when goals can compete

Reason: if the human-facing surface requires codebase knowledge, the user can approve a direction they do not actually understand.

Example issue:
> Important: Layer 1 says "modify `auth_middleware.py:47` before `authenticate()` returns." This is implementation detail, not approval language. Replace it with the user-visible behavior: "expired sessions return Unauthorized instead of Not Found."

### 2. Cold-start executability

Check whether a fresh agent can execute the first task without the planning conversation.

Block if:
- the first task does not name exact existing files or the module/directory where new files belong
- the first task does not state the first behavior-changing action
- a task uses referents like "the existing approach," "the notifier," or "this logic" without defining them in the plan
- verification is prose instead of an executable command with expected output

Reason: if execution depends on conversation context, the plan transfers the planner's memory rather than durable instructions.

Example issue:
> Critical: Task 1 says "set up the notifier" and "run tests." A fresh agent cannot identify the interface or verification command. State the target module, the behavior to add first, and the exact command, e.g. `pytest tests/notifications/test_webhook.py -v → PASS`.

### 3. Assessment evidence

Check whether the plan proves it was written after situation assessment.

Block if:
- the plan cannot state size, work nature, current state, and decomposition strategy
- a bug-fix plan lacks a named root cause
- the plan names files or patterns that do not exist in the repository evidence read
- user uncertainty such as "maybe," "not sure," or "I think" became committed scope without a verification step or explicit decision

Reason: a plan written before assessment can look structurally correct while targeting the wrong problem.

Example issue:
> Critical: The user said the schema "maybe supports batch updates," but Task 2 assumes batch update support. Add a verification step before dependent tasks or mark the behavior as a decision requiring confirmation.

### 4. Decomposition quality

Check whether tasks are vertical slices that leave the codebase working.

Block if:
- tasks are organized by layer only, such as "create all models" then "create all endpoints"
- the first independently testable behavior appears only in the final task
- any task leaves the codebase in a known broken state
- a task does not either deliver verifiable behavior or resolve a named uncertainty

Reason: horizontal slicing delays learning until late execution, so wrong assumptions compound before the first real test.

Example issue:
> Important: Tasks 1-3 create data structures, infrastructure, and business logic before any end-to-end behavior is testable. Reorder around a walking skeleton that proves one job completion sends one webhook before adding config and retry.

### 5. Risk, boundaries, and revision triggers

For medium/large plans, check whether uncertainty is surfaced as execution controls.

Block if:
- no contingency exists for the riskiest assumption
- no plan revision trigger is named
- no explicit exclusion exists where a reasonable agent could expand scope
- cross-boundary work lacks interface or handoff points
- competing goals exist but no conflict priority states which wins in the Human Review Section

Reason: without explicit controls, the executing agent will make scope and trade-off decisions at the moment of highest uncertainty.

Example issue:
> Important: The plan touches CLI, config, and pipeline behavior but does not define the plugin interface contract before implementation. Add a contract-first task or an explicit handoff point before parallel work begins.

### 6. Concrete Design Sketch

Check whether architecture-sensitive work gives a cold-start executor a concrete code shape before tasks begin.

Block if:
- the plan introduces a new architecture boundary or owner such as a service, store, component boundary, persistence boundary, data owner, public interface, or cross-layer adapter but has no Concrete Design Sketch
- the plan changes an existing architecture boundary, ownership boundary, or cross-layer handoff but has no Concrete Design Sketch
- the plan crosses UI/state/API/persistence, CLI/service/domain, or orchestration/domain-effect boundaries but has no Concrete Design Sketch
- the user or review packet names architecture expectations or dissatisfaction with prior implementation shape but the plan only lists abstract constraints
- the sketch contains only advice such as "keep the architecture clean," "use the right layer," or "reuse existing abstractions" without target shape, boundary surfaces, intended flow, ownership, and a shape to avoid
- the sketch omits any required part: target shape, boundary surfaces, intended flow, ownership, or shape to avoid
- any required part exists only as a label but does not name concrete modules, boundary surfaces, flow edges, owners, or the counterexample flow it is meant to prevent
- tasks contradict the sketch or can execute without realizing any part of it
- the plan includes a sketch but lacks a stop signal for implementation evidence that makes the sketch impossible to follow

Reason: behavior tests can pass while code lands in the wrong layer, so architecture-sensitive plans must transfer the intended code shape instead of relying on the executor's taste.

Example issue:
> Important: The plan says the form should stay presentation-only, but there is no component/state sketch. Add a Concrete Design Sketch showing which component owns save lifecycle, which service performs the API call, and the form shape to avoid.

### 7. Progressive refinement for large plans

Check whether far-future tasks are detailed only when their inputs are already known.

Block if:
- more than three large-plan tasks have step-by-step checklists and later steps depend on outcomes from earlier tasks
- a later task specifies exact implementation steps that could change after a risk-reducing spike
- a plan revision trigger is missing after a task that is expected to reveal constraints

Reason: detailed planning beyond known inputs creates false precision and makes the executing agent follow obsolete steps.

Example issue:
> Important: Task 5 specifies discovery implementation before Task 3 decides subprocess vs. in-process sandboxing. Convert Task 5 to goal-level and add "refine after Task 3" as a revision trigger.

## Severity Calibration

Use these severities:

- **Critical**: the plan can direct execution toward the wrong scope, wrong files, unverifiable completion, or an unapproved user-visible behavior.
- **Important**: the plan is probably executable but carries avoidable rework risk, hidden coupling, or ambiguous trade-offs.
- **Minor**: wording or structure can improve readability without changing execution safety.

Blocking threshold:
- Large plans: fix Critical and Important issues before handoff.
- Medium plans: fix Critical issues before handoff; surface Important issues to the human reviewer.

Stop after three review iterations. If blocking issues remain, surface them to the user instead of continuing the loop [because repeated self-repair can converge on a document that looks cleaner without resolving the underlying uncertainty].

## Output Format

Return:

```markdown
## Plan Assessment
- Size reviewed: Tiny / Small / Medium / Large
- Verdict: Approved / Approved with fixes / Blocked
- Blocking threshold applied: Critical only / Critical + Important
- Context used: plan, review packet items, docs, and code files actually read

## Issues
### Critical
- [file/section]: [issue]
  - Evidence: [quote or observable fact]
  - Why it matters: [execution failure mechanism]
  - Fix: [smallest concrete correction]

### Important
- ...

### Minor
- ...

## Final Check
- Human approval surface: pass/fail
- Cold-start executability: pass/fail
- Assessment evidence: pass/fail
- Decomposition: pass/fail
- Risk controls: pass/fail/not applicable
- Concrete Design Sketch: pass/fail/not applicable
- Progressive refinement: pass/fail/not applicable
```

Do not approve a plan solely because all expected sections exist. Section presence is evidence only when the content performs the job of that section.
