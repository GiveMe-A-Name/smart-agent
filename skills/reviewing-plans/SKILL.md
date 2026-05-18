---
name: reviewing-plans
description: "Evaluate an existing plan document against repository constraints, stated intent, design soundness, internal consistency, executability, and missing risk. TRIGGER when a plan document exists and the task is to review, audit, or judge it. DO NOT TRIGGER when creating a plan, implementing a plan, or responding to existing plan-review feedback."
---

# Reviewing Plans

Review a plan as a decision artifact before implementation. The highest-value findings are alignment failures: the plan implements the wrong thing, violates repository constraints, relies on false technical assumptions, or omits the reasoning needed to know whether the approach still holds.

## Principles

- **Review the plan's decisions, not the surrounding artifact.** A plan is reviewable where it chooses an approach, sequence, assumption, interface, migration path, or verification strategy. Illustrative snippets, example commands, and reference text are findings only when the plan depends on them as design decisions [because plan review and code review have different objects; conflating them produces implementation findings rather than approach findings].
- **Repo alignment outranks plan polish.** A plan that violates documented constraints, established architecture, or existing conventions is not redeemed by good task sequencing [because well-executed implementation of a misaligned plan sends work confidently in the wrong direction].
- **Intent is the anchor for scope.** If the plan states tasks but not why the change exists, the reviewer cannot judge whether the approach solves the right problem. If tasks build more than the stated intent requires, treat that as silent scope expansion.
- **Findings need an anchor and a consequence.** Every issue must name the plan location, the specific wrong assumption or decision, and what breaks if the plan proceeds. A constraint citation is evidence for the problem; it is not the production consequence.
- **Severity reflects implementation risk.** Critical is reserved for wrong intent, documented constraint violation, or an approach-invalidating flaw. Important covers false assumptions, internal contradictions, unhandled real risks, and non-executable verification. Minor covers wording or missing detail that will not change implementation outcome.
- **The verdict is part of the review.** End with `Approved`, `Approved with fixes`, or `Blocked`; without a verdict, the plan author is left to review their own work.

## Constraints

- Do not review a plan that has not been read. If no plan document exists, this skill does not apply.
- Before making a repo-alignment finding, read the relevant repository constraint source and cite it with file/line. Common sources are `CLAUDE.md`, `AGENTS.md`, architecture docs, design docs, package conventions, and existing implementation patterns named by the plan.
- If the plan asserts code behavior, file ownership, API shape, schema shape, or existing convention, read the cited file or mark the assertion as unverified. Do not promote plan text into evidence of repository behavior.
- Keep review independence. User framing, author emphasis, and plan-stated risk priorities are inputs, not conclusions. Findings come from comparing the plan against intent, repository evidence, and the layered checks below.
- Do not rewrite the plan. Give targeted fixes for findings, but do not produce a replacement plan unless the user explicitly asks for one.
- Do not review implementation code except to validate plan assumptions. If the issue is only that an illustrative snippet has a code-style problem and the plan does not depend on it, omit it.
- If a Critical issue is present, the verdict must be `Blocked`. `Approved with fixes` is for non-critical issues that do not invalidate the approach.

## Layered Review

Review from the highest-impact layer downward. Stop cataloging lower-layer polish when an upper-layer issue already blocks the plan.

| Layer | Question | Signals |
|-------|----------|---------|
| **0: Repo Alignment** | Does the plan conflict with documented constraints or existing architecture? | Violates `CLAUDE.md`, `AGENTS.md`, architecture docs, package boundaries, build/test conventions, or established implementation patterns. |
| **1: Intent & Design** | Is this the right approach for the stated intent? | Intent is missing or mismatched; scope is silently larger than claimed; cliff-edge risks are late or unrecoverable; shared-infrastructure blast radius is unstated; constraints can conflict without priority; technical assumptions are false or unverified. |
| **2: Internal Consistency** | Do the tasks cohere and collectively achieve the goal? | Tasks contradict each other; dependency order is wrong; interfaces/types do not line up; verification is claimed before the code can satisfy it; vague qualities like "scalable" lack an operational definition. |
| **3: Executability** | Can an implementer follow the plan without getting stuck? | File paths are wrong; commands are not executable; expected outputs are absent; tasks depend on later tasks; no revision trigger exists for uncertain branches. |
| **4: Completeness** | Are important gaps acknowledged? | Major design decisions lack alternatives/reasoning; edge cases or failure modes required by the approach are missing. |

## Review Checks

- **Intent check:** The first substantive section should say why the change exists. If the plan opens with file edits or task lists and no intent, flag it as Important.
- **Scope check:** Compare stated intent against actual tasks. New abstractions, interfaces, migrations, or shared infrastructure outside the stated intent are Layer 1 findings.
- **Risk check:** If the plan names surface risks while a deeper structural risk is visible, flag the misplaced risk model. Example: "column names might not match" is not the main risk when the proposed idempotency check can permanently skip partial writes.
- **Cliff-edge check:** Identify tasks where failure invalidates the whole approach. If such a task appears late, has no fallback, or lacks a revision trigger, flag it.
- **Constraint-priority check:** If two stated constraints can conflict, the plan must name which wins. Otherwise the implementer is forced to make a design decision during execution.
- **Blast-radius check:** For shared infrastructure changes, state what fails beyond the target feature and at what scale. A base-class bug affecting every consumer is materially different from a bug isolated to one feature.
- **Decision-traceability check:** Architectural, interface, schema, migration, and dependency choices need alternatives considered or a reason the chosen path follows existing constraints. Missing rationale is Important when later evidence could invalidate the choice.
- **Verification check:** "Run tests" or "verify behavior" is not executable. A useful verification step names a command, target, expected result, or observable acceptance condition.

## Output

Use `templates/review-output.md` for the full structure when writing a complete review.

Required sections:

- **Plan Assessment:** one or two sentences restating the plan's intent, then Layer 0 and Layer 1 assessment.
- **Issues:** grouped by severity. Omit a severity group only when there are no findings in that group.
- **Verdict:** `Approved`, `Approved with fixes`, or `Blocked`.

Each issue must include:

- title
- severity
- layer
- plan location
- what is wrong
- why it matters: concrete consequence, not just a rule citation
- fix: targeted corrective action when the issue does not already determine the required change

## Stop And Revise

Stop and revise the review if any condition is true:

- Findings were written before reading the plan document.
- A Layer 0 finding lacks a repository source citation from this session.
- A plan assertion about code behavior was accepted without reading the referenced code or marking it unverified.
- Every finding is Layer 3 or Minor while Layers 0-1 were not checked against repo constraints and intent.
- A Critical finding lacks a concrete consequence, or its `Why` field only says which rule is violated.
- All findings are Critical without severity calibration against actual implementation risk.
- The review describes issues as "could cause problems" or "may be risky" without naming what fails and where.
- The verdict is `Approved with fixes` while any Critical issue remains.
- The review ends without a verdict.
