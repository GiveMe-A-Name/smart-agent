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
- no plan document exists yet
- the task is to respond to plan review feedback that already exists

## Capability Boundary

This skill owns: reading plan documents against repo constraints and plan quality criteria, identifying issues across all review layers, calibrating severity, and delivering a structured verdict.

It does NOT:
- rewrite or fix the plan
- review code implementation — only the plan document
- approve implementation details that belong to execution judgment

## Invariants

**Read context before reviewing.** A plan review without reading CLAUDE.md, architecture docs, and any files the plan references is a false review. Every Layer 0 finding requires a named source: the specific constraint the plan violates, with a file:line citation.

**Upper-layer problems outweigh lower-layer problems.** A plan that violates repo constraints does not become acceptable because its tasks are well-sequenced. Work top-down — if a Layer 0 problem is severe enough to block implementation, surface it immediately rather than cataloging Layer 3 issues in a plan that may need to be rewritten.

**Every issue needs: plan location, what is wrong, why it matters.** Findings like "the plan should verify this" or "this could cause problems" are not actionable. Name the specific section or line, the specific problem, and the specific consequence.

**Severity must reflect actual implementation risk.** Critical means the plan would implement the wrong thing, violate a documented constraint, or have a fundamental flaw that invalidates the approach — not style, not missing detail. Inflating severity makes all feedback untrustworthy.

**Give a clear verdict.** Every review ends with: Approved / Approved with fixes / Blocked. An ambiguous conclusion defers the decision to the plan author, which is the reviewer's job.

## The Layered Review Model

Work from the most consequential question down.

| Layer | Key question | Critical signals |
|-------|-------------|-----------------|
| **0: Repo Alignment** | Does this plan conflict with documented constraints? | CLAUDE.md, AGENTS.md, architecture docs, existing conventions — any violation here means wrong implementation. |
| **1: Intent & Design** | Does the plan know why it exists, and is this the right approach? | Explicit intent statement present (why, not just what). Scope matches claims. Cliff-edge risks identified — tasks where failure invalidates the whole approach, not just delays it. Constraint conflicts have a stated priority. Technical assumptions stated and plausible. No silently larger change than advertised. |
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

**Decision traceability:** For each major design choice, does the plan name the alternatives considered and why they were rejected? If not, the reviewer cannot assess whether the reasoning still holds. Flag as Important for any architectural or interface decision with no stated rationale.

**Revision trigger check:** Does the plan state conditions under which the approach should be revisited? (e.g., "if Task 2 reveals X, redesign the query path"). For medium/large plans, absence of revision triggers is an Important finding — the plan assumes perfect foresight.

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

- you are writing findings without having read CLAUDE.md and architecture docs — repo constraint violations are the highest-value finding and cannot be detected without this context
- every finding is Layer 3 (executability detail) and you found nothing at Layers 0–2 — either the plan is genuinely sound at the higher layers, or you didn't check intent, decision traceability, or cliff-edge risks
- a Critical finding lacks a named source (specific file:line in repo docs) — a Critical finding without evidence is speculation
- all findings are labeled Critical — severity inflation makes all feedback untrustworthy
- findings are described as "could cause issues" or "may be a problem" without naming the specific consequence — vague risk statements are not actionable
- the review ends without a verdict — an open-ended summary is not a review
- you are reviewing plan structure and style while ignoring whether the plan would build the right thing
- the plan has no intent statement and you did not flag its absence — a plan that only states "what" without "why" cannot be evaluated for alignment with actual need

## Completion Criteria

- [ ] CLAUDE.md, architecture docs, and plan-referenced code files were read before reviewing.
- [ ] The review covers repo alignment (Layer 0) and intent/design (Layer 1) — not just internal structure and format.
- [ ] Every Critical finding cites a specific repo constraint (file:line) that the plan violates.
- [ ] Every issue includes plan location, what is wrong, and why it matters, with honest severity calibration.
- [ ] Intent is present: the plan states why the change is being made, not just what tasks to do. If absent, flagged.
- [ ] Key decisions were checked for decision traceability — alternatives named and reasoning present.
- [ ] For medium/large plans: cliff-edge risks and revision triggers checked; absence noted if missing.
- [ ] A clear verdict is present (Approved / Approved with fixes / Blocked).

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the review is real.

Did I mistake reading the plan document for sufficient context — without reading the repo constraints it should align with?

Am I exiting because the review is genuinely complete, or because the findings list now looks thorough enough?
