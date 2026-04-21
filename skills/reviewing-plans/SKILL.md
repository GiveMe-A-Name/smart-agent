---
name: reviewing-plans
description: "Review an existing plan document for repo alignment, design soundness, internal consistency, and executability. TRIGGER when: a plan document exists and the task is to evaluate it. DO NOT TRIGGER when: writing a new plan (use writing-plans), or responding to plan review feedback."
---

# Reviewing Plans

A plan review that misses repo constraint violations is worse than no review — it sends implementers confidently in the wrong direction. The highest-value findings are not style or structure problems — they are alignment failures: the plan implements the wrong thing, violates documented conventions, or makes technical assumptions that are false.

## Trigger Logic

**Invocation default**: Skipping structured review lets flawed plans reach implementation. The cost of unnecessary review is a short overhead. The cost of missing it is an implementation that violates architecture constraints or has to be undone.

Use when:
- a plan document exists and the task is to evaluate it
- completing a writing-plans cycle and review is the gate before implementation
- another agent has produced a plan that needs independent verification

Do not use when:
- no plan document exists yet — write the plan first
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
| **1: Design Soundness** | Is this the right approach? | Scope matches what the plan claims. Real risks identified, not surface-level ones. Technical assumptions stated and plausible. No silently larger change than advertised. |
| **2: Internal Consistency** | Do the tasks cohere? | No contradictions between tasks. Dependencies respected. Verification criteria achievable at the point they're claimed. Interfaces/types consistent across tasks. |
| **3: Executability** | Can an implementer follow this without getting stuck? | Verification steps are commands with expected outputs, not prose. Tasks don't depend on later tasks. File paths accurate. Each task leaves the codebase in a working state. |

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

## Output Format

See `templates/review-output.md` for the full output structure.

**Required sections:** Plan Assessment (Layer 0–1) · Issues (by severity) · Verdict

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
- every finding is Layer 3 (executability detail) and you found nothing at Layers 0–2 — either the plan is genuinely sound at the higher layers, or you didn't look hard enough
- a Critical finding lacks a named source (specific file:line in repo docs) — a Critical finding without evidence is speculation
- all findings are labeled Critical — severity inflation makes all feedback untrustworthy
- findings are described as "could cause issues" or "may be a problem" without naming the specific consequence — vague risk statements are not actionable
- the review ends without a verdict — an open-ended summary is not a review
- you are reviewing plan structure and style while ignoring whether the plan would build the right thing

## Completion Criteria

- [ ] CLAUDE.md, architecture docs, and plan-referenced code files were read before reviewing.
- [ ] The review covers repo alignment (Layer 0) and design soundness (Layer 1) — not just internal structure and format.
- [ ] Every Critical finding cites a specific repo constraint (file:line) that the plan violates.
- [ ] Every issue includes plan location, what is wrong, and why it matters, with honest severity calibration.
- [ ] A clear verdict is present (Approved / Approved with fixes / Blocked).

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the review is real.

Did I mistake reading the plan document for sufficient context — without reading the repo constraints it should align with?

Am I exiting because the review is genuinely complete, or because the findings list now looks thorough enough?
