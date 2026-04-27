---
name: reviewing-plans
description: "Review an existing plan document for repo alignment, intent clarity, design soundness, internal consistency, executability, and completeness. TRIGGER when: a plan document exists and the task is to evaluate it. DO NOT TRIGGER when: writing a new plan, or responding to plan review feedback."
---

# Reviewing Plans

A plan review that misses repo constraint violations is worse than no review — it sends implementers confidently in the wrong direction. The highest-value findings are not style or structure problems — they are alignment failures: the plan implements the wrong thing, violates documented conventions, makes false technical assumptions, or omits the reasoning behind key decisions so no one can tell whether the approach still holds.

## Trigger Logic

**Invocation default**: Skipping structured review lets flawed plans reach implementation. The cost of unnecessary review is a short overhead. The cost of missing it is an implementation that violates architecture constraints or has to be undone.

Use when:
- a plan document exists and the task is to evaluate it
- another agent has produced a plan that needs independent verification

Do not use when:
- no plan document exists yet [because reviewing a plan that doesn't exist converts the review into plan-writing, which is a different skill]
- the task is to respond to plan review feedback that already exists [because responding to review feedback is implementation work — conflating it with reviewing produces an advocate for the implementation rather than an independent evaluator]

## Capability Boundary

This skill owns: reading plan documents against repo constraints and plan quality criteria, identifying issues across all review layers, calibrating severity, and delivering a structured verdict.

It does NOT:
- rewrite or fix the plan
- review code implementation — only the plan document (for illustrative code snippets inside the plan itself, see Artifact boundary below)
- approve implementation details that belong to execution judgment

**Artifact boundary:** A plan is a design intent document. Its reviewable substance is the design decisions it makes — the approach, task sequence, stated assumptions, and architectural choices. Material inside the plan that is not itself a design decision (illustrative code, example commands, reference snippets) is context for understanding the design, not a claim to evaluate. The test: is this finding about a decision the plan is making? If not, it is not a plan-level finding. [Because plan review and code review have different objects — conflating them produces findings that belong to implementation, not to whether the approach is sound.]

**Reviewer independence:** This review is an independent assessment of the plan against repo constraints and quality criteria. What the requester emphasizes in the prompt, what the plan's author considers the primary risk, and what feels most salient in the moment are inputs — not conclusions. Findings emerge from the layered review of the plan document, not from the framing that arrived with it. [Because a reviewer who adopts the requester's prioritization is not an independent assessor — it is validating a frame rather than evaluating a plan.]

## Invariants

**Read context before reviewing.** A plan review without reading CLAUDE.md, architecture docs, and any files the plan references is a false review. Every Layer 0 finding requires a named source: the specific constraint the plan violates, with a file:line citation. [Because Layer 0 violations are the highest-value finding class — and they are invisible without reading the repo constraints the plan should align with.]

**Upper-layer problems outweigh lower-layer problems.** A plan that violates repo constraints does not become acceptable because its tasks are well-sequenced. Work top-down — if a Layer 0 problem is severe enough to block implementation, surface it immediately rather than cataloging Layer 3 issues in a plan that may need to be rewritten. [Because detailed executability polish on a misaligned plan sends implementers confidently in the wrong direction.]

**Every issue needs: plan location, what is wrong, why it matters.** Findings like "the plan should verify this" or "this could cause problems" are not actionable. Name the specific section or line, the specific problem, and the production consequence — what breaks if the plan proceeds, at what scale, and whether recovery is possible. Constraint citations ("violates AGENTS.md:4") are evidence for "what is wrong" — put them there. "Why it matters" must state the consequence: what fails in production, at what scale. "A bug in this shared base class fails every API endpoint simultaneously with no partial rollback path" is a consequence; "violates AGENTS.md:4" is not. [Because an implementer cannot evaluate the severity or urgency of a finding without knowing where the issue is and what specifically breaks if it isn't addressed.]

**Severity must reflect actual implementation risk.** Critical means the plan would implement the wrong thing, violate a documented constraint, or have a fundamental flaw that invalidates the approach — not style, not missing detail. Inflating severity makes all feedback untrustworthy. [Because when everything is Critical, implementers start discounting all findings — the signal is destroyed by the noise.]

**Give a clear verdict.** Every review ends with: Approved / Approved with fixes / Blocked. An ambiguous conclusion defers the decision to the plan author, which is the reviewer's job. [Because listing considerations without a verdict shifts the decision to the person least able to evaluate the findings independently — the author who wrote the plan.]

## The Layered Review Model

Work from the most consequential question down.

| Layer | Key question | Critical signals |
|-------|-------------|-----------------|
| **0: Repo Alignment** | Does this plan conflict with documented constraints? | CLAUDE.md, AGENTS.md, architecture docs, existing conventions — any violation here means wrong implementation. |
| **1: Intent & Design** | Does the plan know why it exists, and is this the right approach? | Explicit intent statement present (why, not just what). Scope matches claims. Cliff-edge risks identified — tasks where failure invalidates the whole approach, not just delays it. Blast-radius of shared infrastructure changes stated — components outside the plan's scope that could be affected are named. Constraint conflicts have a stated priority. Technical assumptions stated and plausible. No silently larger change than advertised. |
| **2: Internal Consistency** | Do the tasks cohere, and do they collectively achieve the stated goal? | No contradictions between tasks. Dependencies respected. Verification criteria achievable at the point claimed. Interfaces/types consistent. Sub-tasks collectively derive the stated goal — not just look related. No vague terms ("flexible", "scalable") used without an operational definition. |
| **3: Executability** | Can an implementer follow this without getting stuck? | Verification steps are commands with expected outputs. Tasks don't depend on later tasks. File paths accurate. Each task leaves the codebase in a working state. Revision triggers stated — conditions under which the approach should change. |
| **4: Completeness** | Are there gaps the plan doesn't acknowledge? | Key decisions carry explicit reasoning: alternatives considered and why rejected. Counter-intuitive edge cases or failure modes that the plan's approach must survive. |

**Depth calibration by risk:**

| Plan characteristics | Depth |
|---------------------|-------|
| Architecture change, new system boundary, multi-component impact | Maximum — verify every constraint and assumption |
| New feature, new interface, schema change | High — full Layer 0–3 review |
| Bug fix with clear, bounded scope | Medium — focus on alignment and consistency |
| Refactor within existing pattern | Light — verify scope and key assumptions |

**Severity calibration:**

| Severity | What qualifies |
|----------|---------------|
| Critical | Plan would implement the wrong thing, violate a documented constraint, or have a flaw that invalidates the approach | 
| Important | Internal inconsistency, false technical assumption, real risk not identified, verification step not executable |
| Minor | Wording, missing detail that will not cause implementation problems |

## Decision Signals

**What context to read:** Before reviewing, identify what the plan references and read it. At minimum: CLAUDE.md, any architecture docs named in the plan, any code files the plan assumes behavior from. A plan review without this context cannot catch Layer 0 problems.

**Scope creep signal:** Compare what the plan's intent statement claims against what the tasks actually build. If the tasks introduce abstractions, interfaces, or patterns not mentioned in the intent, name the gap. Silent scope expansion is a Layer 1 finding.

**Risk misidentification:** If the plan's stated risks are "column names might not match" or other surface-level concerns while a deeper structural risk is present (e.g., warm-up semantics, partial-write idempotency, downstream contract drift), that is an Important finding — the plan will be executed with the wrong concerns prioritized.

**Verification step quality:** A step that says "run the tests" or "verify behavior" is not executable. A step that says `pytest tests/foo.py::test_bar -v → PASS` is. Flag non-executable verification as Important.

**Internal contradiction test:** If two tasks would produce incompatible outcomes (e.g., Task 2 expects `{"factors": []}` but Task 5 emits a different envelope structure), that is an Important finding regardless of which task is "correct."

**Intent check:** The plan's first substantive content should be an intent statement — why this change is being made. If the plan opens directly with tasks or file changes without a preceding intent, flag it as Important. Without intent, the reviewer cannot assess whether the approach serves the actual need.

**Cliff-edge identification:** Find tasks where failure invalidates the entire plan, not just that task. If such a task exists late in the sequence or has no recovery path, flag it as Important — execution will stall or backtrack at the worst point.

**Constraint conflict priority:** List the plan's stated constraints. If any two can conflict (e.g., "must not break existing API" and "must extend the same endpoint"), check whether the plan states which wins. An unstated priority forces the implementer to decide at execution time.

**Blast-radius check:** When a plan modifies shared infrastructure — base classes, middleware, shared schemas, global configuration, or any code used by components outside the plan's stated scope — explicitly answer: if this task has a subtle bug, what fails beyond this feature and at what scale? A targeted change where a bug only breaks the feature being built has blast-radius 0. A change to a shared base class where a bug takes down every consumer simultaneously has blast-radius proportional to the number of dependents. State the scale in the finding's Why field. Do not let a constraint citation substitute for this — "violates AGENTS.md:4" describes the rule; the blast-radius describes the stakes.

**Decision traceability:** For each major design choice, does the plan name the alternatives considered and why they were rejected? If not, the reviewer cannot assess whether the reasoning still holds. Flag as Important for any architectural or interface decision with no stated rationale.

**Revision trigger check:** Does the plan state conditions under which the approach should be revisited? (e.g., "if Task 2 reveals X, redesign the query path"). For medium/large plans, absence of revision triggers is an Important finding — the plan assumes perfect foresight.

## Verification Approach

This skill is artifact-type: the review document is the artifact. Completion is verified by self-checking the produced review against the Completion Criteria checklist. The evidence is the review document itself — not the agent's assessment of whether the review feels thorough.

A review that satisfies the checklist structurally is not necessarily complete. The checklist items are proxies. The underlying test: could an implementer read this review and know exactly what to fix, where to fix it, and whether the plan is safe to execute?

## Output Format

See `templates/review-output.md` for the full output structure.

**Required sections:** Plan Assessment (Layers 0–1) · Issues (by severity) · Verdict

**For each issue:**
- Named title
- Severity
- Location in plan (file:line)
- What is wrong
- Why it matters
- Recommendation (if not obvious)

**Verdict options:** `Approved` / `Approved with fixes` / `Blocked`

## Failure Signals

Stop and reassess if:

- you are writing findings without having read CLAUDE.md and architecture docs [because Layer 0 violations are the highest-value class of finding and cannot be detected without reading the constraints the plan should align with — a review without this context is a false review]
- every finding is Layer 3 (executability detail) and you found nothing at Layers 0–2 — either the plan is genuinely sound at the higher layers, or you didn't check intent, decision traceability, or cliff-edge risks
- a Critical finding lacks a named source (specific file:line in repo docs) — a Critical finding without evidence is speculation
- all findings are labeled Critical — severity inflation makes all feedback untrustworthy
- findings are described as "could cause issues" or "may be a problem" without naming the specific consequence — vague risk statements are not actionable
- a Critical finding's Why field contains only a constraint citation ("violates AGENTS.md:4", "not permitted by CLAUDE.md:12") — a constraint citation is evidence for What, not a consequence for Why; state what breaks in production if the plan proceeds [because an implementer who reads only a constraint citation cannot assess the severity or urgency of the finding without knowing what specifically fails and at what scale]
- the review ends without a verdict — an open-ended summary is not a review
- you are reviewing plan structure and style while ignoring whether the plan would build the right thing
- the plan has no intent statement and you did not flag its absence [because a plan that only states "what" without "why" cannot be evaluated for alignment with actual need — and silently accepting the absence makes the review an evaluation of internal consistency on a plan that may be solving the wrong problem]

## Completion Criteria

- [ ] CLAUDE.md, architecture docs, and plan-referenced code files were read before reviewing.
- [ ] The review covers repo alignment (Layer 0) and intent/design (Layer 1) — not just internal structure and format.
- [ ] Every Critical finding cites a specific repo constraint (file:line) that the plan violates.
- [ ] Every Critical finding states the production consequence — what breaks if the plan proceeds, at what scale. A constraint citation alone does not satisfy this criterion.
- [ ] Every issue includes plan location, what is wrong, and why it matters, with honest severity calibration.
- [ ] Intent is present: the plan states why the change is being made, not just what tasks to do. If absent, flagged.
- [ ] Key decisions were checked for decision traceability — alternatives named and reasoning present.
- [ ] For medium/large plans: cliff-edge risks and revision triggers checked; absence noted if missing.
- [ ] A clear verdict is present (Approved / Approved with fixes / Blocked).

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the review is real.

This check exists because a review that looks thorough is not the same as a review that is thorough. The agent cannot reliably distinguish them from the inside — a complete findings list feels the same as a hollow one. This section externalizes that checkpoint.

Did I mistake reading the plan document for sufficient context — without reading the repo constraints it should align with?

Am I exiting because the review is genuinely complete, or because the findings list now looks thorough enough?

Completion-faking signals specific to plan review — stop if any apply:

- All findings are Layer 3 (executability) and nothing surfaced at Layers 0–1, not because the plan is genuinely sound at those layers, but because checking Layer 0 requires reading repo docs and that work was skipped
- A Critical finding is present but labeled Important or Minor to appear balanced — severity was softened to avoid delivering a harsh conclusion
- Findings use hedged language ("could cause issues", "may be a problem") without naming the specific consequence — they look like findings but cannot be acted on
- A Critical finding's Why contains only a constraint citation ("violates X:N") and no production consequence — the rule was cited but the stakes were not described; what breaks in production and at what scale is missing
- The verdict is "Approved with fixes" when a Critical finding is present — a Critical finding means the approach or a documented constraint is violated, which qualifies as Blocked
- The plan has no intent statement and it wasn't flagged — the review evaluated internal consistency on a plan that may be solving the wrong problem
