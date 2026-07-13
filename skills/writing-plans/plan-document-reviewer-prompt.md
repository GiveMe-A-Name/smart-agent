# Independent Plan Review Guidance

Use this guidance when a plan requires independent review. Review whether a fresh human can approve the right decision and a fresh agent can execute safely; do not reward section count or template completeness.

## Review Contract

The plan has up to three reading surfaces:

- **Decision Brief:** outcome approval; required for every plan
- **Design Review:** technical design approval; present only when its gate is triggered
- **Execution Plan:** repository evidence, exact work, verification, risks, and handoff

The review identifies concrete failure mechanisms and the smallest correction. It is not a request to rewrite the plan in the reviewer's preferred style.

## Required Context

Before reviewing:

- read the complete plan
- if invoked as a sub-agent, use only the stated intent, constraints, plan, docs, and files in the review packet
- read repository instructions and architecture docs named by the plan
- read files cited as current-state evidence or modification targets
- record missing referenced files as issues rather than inferring replacements

Do not infer hidden goals or conversation history. Missing context is a finding only when the plan needs it for approval, execution, or verification.

## Review Checks

### 1. Decision Brief: can the human decide?

Check whether a reader without codebase knowledge can state the intended direction, observable outcome, and scope.

Block when:

- the first substantive section after an optional document title is not Decision Brief
- Intent, Outcome, or Scope is absent
- Intent and Outcome paraphrase each other instead of separating problem/value from final behavior
- Outcome contains tasks, files, commands, or internal implementation names
- Scope omits an adjacent behavior whose preservation is approval-critical
- multiple visible outcomes, a public contract change, compatibility expectation, migration, restored behavior, or competing interpretations exist but Visible Changes is absent or incomplete
- Approval Needed is absent despite a material human choice, or appears as an empty/`None` placeholder
- Primary Risk appears for ordinary implementation difficulty, or a direction-changing uncertainty is hidden only in technical tasks
- task summaries, task ordering, file maps, or technical assessment leak into the brief
- the same fact is repeated across multiple fields without serving a different approval question

Do not block because an optional field is absent when its trigger is absent.

Example issue:

> Important: Intent and Outcome both say “add CSV export.” Rewrite Intent to state why users need it and Outcome to state the observable download behavior; delete the duplicate sentence.

### 2. Design Review: is it necessary and sufficient?

First apply the gate. Design Review is required only when evidence shows a material public contract, persisted schema, event contract, compatibility, security, data integrity, recovery, migration, rollback, service/process/persistence/trust/external-dependency boundary, disputed responsibility/dependency choice, or explicit user design request.

Block when:

- a gate trigger exists but no Design Review exposes the affected choice
- Design Review exists without a trigger and merely restates approach or tasks
- it does not state the proposed shape, material alternative or binding constraint, and evidence-based rationale
- it uses a representation unrelated to the decision, such as an architecture diagram for a contract-only change
- it repeats the same information in a diagram, topology, responsibility list, flow, and boundary list
- it invents alternatives unsupported by repository evidence or user direction
- it contains exact file edits, task sequence, production code, or private helper detail
- tasks contradict the approved contract, boundary, dependency, failure policy, migration, or rollout without a stop signal

Do not require a fixed set of fields. One diagram, behavior matrix, contract diff, pseudocode block, or migration sequence may be sufficient.

Example issue:

> Important: The plan adds a nullable API field but Design Review contains a component diagram and no compatibility statement. Replace the diagram with a Before/After contract diff and state how existing clients remain valid.

### 3. Cold-start executability

Block when:

- the first task does not name exact existing files or the module/directory for new files
- the first task does not state the first behavior-changing or uncertainty-resolving action
- a task depends on undefined referents such as “the existing approach” or “the notifier”
- verification is prose rather than a command plus expected output

Reason: an execution plan that depends on conversation memory is not a durable handoff.

### 4. Assessment evidence

Block when:

- size, work nature, current state, observable done state, final verification, or decomposition strategy cannot be established
- the assessment uses verification commands as the done state instead of describing the behavior, contract, or state those commands must prove
- a bug fix lacks a named root cause
- a refactor lacks caller/dependency evidence
- a feature cites no existing pattern and does not state that none exists
- cited files or patterns do not exist in the evidence read
- hedged user suggestions become committed scope without a decision or verification task

### 5. Decomposition quality

Block when:

- tasks are grouped only by layers or file categories
- the first independently testable behavior appears only at the end
- a task leaves the repository intentionally broken
- a task neither delivers verifiable behavior nor resolves named uncertainty
- task boundaries have no independent verification signal

### 6. Risk controls and boundaries

For Medium/Large plans, block when:

- the riskiest assumption has no response classified by approval impact: a plan-internal contingency/revision trigger or an approval-changing stop signal
- a contingency or revision trigger would change the approved Decision Brief or Design Review instead of stopping for human decision
- the same risk condition is repeated as both a contingency/revision trigger and a stop signal
- a reasonable execution agent could expand scope because exclusions are missing
- cross-boundary work omits inputs, outputs, errors, owner, or handoff point
- a conflict affecting outcome, compatibility, risk, or scope is not surfaced in Approval Needed
- implementation evidence could invalidate approved design semantics but no stop signal requires re-approval

### 7. Progressive refinement

For Large plans, block when:

- more than the first two or three tasks have detailed checklists while later inputs depend on early results
- later tasks prescribe implementation that a risk-reducing spike may invalidate
- no revision trigger follows a task expected to reveal architecture or contract constraints

### 8. Cross-surface duplication

Check the plan as one document:

- outcome and scope belong in Decision Brief
- architecture choices and alternatives belong in Design Review
- files, commands, sequencing, and detailed risks belong in Execution Plan
- observable completion belongs in the assessment; the complete final command set belongs only in Final Verification
- a risk condition has one canonical response location based on whether handling it changes the approved surface

Report duplication when deleting one copy would preserve all meaning. Repetition is allowed only when an execution task references an approved invariant without re-explaining its rationale.

## Severity Calibration

- **Critical:** can direct execution toward wrong scope, wrong files, unverifiable completion, unsafe behavior, or an unapproved visible/design change
- **Important:** executable but carries avoidable rework, hidden coupling, approval ambiguity, or material reading burden
- **Minor:** wording or navigation issue without execution or approval risk

Blocking threshold:

- Large: fix Critical and Important issues before handoff
- Medium: fix Critical issues; surface Important issues to the human reviewer

For Medium plans, independent review is required by default. It may be skipped only when the plan affects one domain without changing a public contract or service/process/persistence/trust/external-dependency/cross-team boundary, has no unresolved risk capable of changing approved outcome/scope/design, needs fewer than two contingencies, and names a repository-verified existing pattern. If any condition is false or unknown, review is required.

Stop after three review iterations. If blocking issues remain, surface them rather than polishing the document indefinitely.

## Output Format

```markdown
## Plan Assessment
- Size reviewed: Tiny / Small / Medium / Large
- Verdict: Approved / Approved with fixes / Blocked
- Blocking threshold: Critical only / Critical + Important
- Context used: [plan, packet items, docs, and files actually read]

## Issues
### Critical
- [section]: [issue]
  - Evidence: [quote or observable fact]
  - Why it matters: [approval or execution failure mechanism]
  - Fix: [smallest concrete correction]

### Important
- ...

### Minor
- ...

## Final Check
- Decision Brief: pass/fail
- Design Review gate: pass/fail/not applicable
- Design Review content: pass/fail/not applicable
- Cold-start executability: pass/fail
- Assessment evidence: pass/fail
- Decomposition: pass/fail
- Risk controls: pass/fail/not applicable
- Progressive refinement: pass/fail/not applicable
- Cross-surface duplication: pass/fail
```

Do not approve a plan solely because expected headings exist. A section passes only when its content supports the reader's decision or action.
