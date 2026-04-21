---
name: brainstorming
description: "Explore design options for a feature, workflow, or component before any spec exists. TRIGGER when: idea is open-ended, no committed spec or design doc exists yet. DO NOT TRIGGER when: a spec already exists and the task is implementation or planning."
---

# Brainstorming Ideas Into Designs

Turns an idea into a validated, written spec through structured collaborative dialogue. The output is a committed design document — not just a conversation.

<HARD-GATE>
Do NOT write code, scaffold a project, or invoke any implementation skill until the user has approved the design, the spec has passed review, and the user has reviewed the committed spec.
</HARD-GATE>

## Trigger Signals

**Invocation default**: Skipping design when it is needed converts unresolved questions into implementation mistakes — wrong interfaces, missing constraints, or structure that has to be undone. The cost of an unnecessary design session is a short conversation. The cost of missing it is building the wrong thing. Invoke this skill whenever no committed spec exists and the work involves decisions about what to build or how it should work.

This skill should load when the user's message implies design is needed before implementation, even if they do not explicitly ask for a spec.

Common signals:
- They have an idea, feature, workflow, or product change, but key details are still undecided
- They ask to brainstorm, think through, scope, design, compare options, or figure out the best approach
- They ask implementation-first questions, but the real blocker is still unresolved design or unclear requirements
- They describe a desired solution before clarifying the problem, users, constraints, tradeoffs, or success criteria
- They are choosing between architectures, UX flows, API shapes, or system boundaries
- They want to start building something new in an existing codebase, but the shape of the change is not yet agreed

Do not use this skill when:
- a committed spec already exists, the approach and interfaces are decided, and the remaining step is writing code against that spec
- the question is a factual lookup or a clarification with no decisions to make about shape, scope, tradeoffs, or approach

## Capability Boundary

This skill covers design-phase work: shaping an idea into a spec with enough fidelity to support implementation planning.

This skill does NOT own:
- Implementation planning or test design
- Sub-project decomposition beyond identifying that it's needed
- Designing a scope the user hasn't yet agreed to decompose

When scope is too large and the user isn't ready to decompose, name that blocker rather than designing around it.

## Invariants

- One question per message — no exceptions
- Always propose 2–3 approaches before presenting a design — unless the user has explicitly named an approach and said they do not need alternatives
- Every project gets a design — simple ones get a short one, not no design
- Spec is committed to git before the user reviews it
- Verbal approval of a design in conversation is not the same as approval of the written spec
- No unrelated refactoring — only improvements that directly serve the current goal

## Decision Signals

**Scope flagging:** If the request describes multiple independent subsystems, flag decomposition immediately. Do not ask detail questions about a project that first needs to be broken apart. Help the user identify the independent pieces, their order, and then brainstorm only the first sub-project through the normal flow.

**Question ordering:** Start with purpose and constraints — what problem, for whom, with what limits. Move to success criteria and edge cases only after the problem is clear. Ask about technical preferences only when they would constrain the design direction, not as an opener. Starting with "what technology do you want to use?" before understanding the problem is a failure.

**Blind spot injection:** At session start, read `skills/brainstorming/references/blind-spots.md`. Select 2–3 blind spots most relevant to this topic — not the most obvious ones, but those where the user's framing reveals the strongest implicit assumptions. Inject each as a dedicated standalone question at the moment it becomes most relevant during the session, not all at the start. Skip any blind spot the user has already addressed unprompted. Each injected blind spot question counts as one of the one-question-per-message turns.

**Project context:** Explore files, docs, and recent commits before asking questions. Questions you could answer by reading the codebase waste the stakeholder's time. Follow existing patterns. Where existing code has problems that affect the work (overgrown files, tangled responsibilities), include targeted improvements as part of the design — not as unrelated refactoring.

**Technical feasibility spike:** Pause design and spike when: an approach depends on undocumented or ambiguous third-party behavior; performance characteristics are central to the decision and cannot be estimated; or the integration seam between two systems is poorly understood. A spike is not implementation — it answers one design question. Set a strict timebox (30 minutes to 2 hours). A spike that invalidates an approach is a success.

**External research:** When an approach depends on external libraries, services, or APIs: verify the dependency actually supports the required capability before committing the design to it; check for known limitations, breaking changes, deprecation signals, and maintenance health. If evaluation requires sustained multi-source research, pause design, gather evidence, then return with facts.

**Approach recommendation:** Prefer approaches that meet constraints with minimal dependencies. When options are equivalent in outcome, prefer the simpler one. Lead with your recommendation and reasoning — don't just list options and make the user choose without guidance.

**Approach comparison:** Compare 2–3 approaches against consistent dimensions so the user can evaluate tradeoffs. Choose dimensions that matter for this specific decision: complexity and maintenance burden, risk profile and recoverability, dependency footprint, reversibility. Present as a table when comparing 3+ approaches across 3+ dimensions. Two approaches differing in one dimension do not need a table.

**Enough questions:** Move to approach proposals when you can answer: what is it for, who uses it, what are the constraints, and what does success look like? If you cannot answer all four after 5–6 questions, move to proposals anyway — missing information is better surfaced through option trade-offs than through more questions.

**Design sufficiency:** The design is complete when you can answer: (1) what units exist and what are their interfaces, (2) how data flows through the system, (3) what error cases exist, (4) how each unit is tested. Any gap means more work. For narrow, well-understood requests, a brief bullet-point summary covering these four dimensions is sufficient. Write the spec once the user has approved each major section — do not write to discover gaps.

**Design isolation:** Break the system into units each with one clear purpose, well-defined interfaces, and independent testability. A unit you cannot understand or test independently needs its boundaries refined.

**Spec review:** Dispatch reviewer using `skills/brainstorming/templates/spec-document-reviewer-prompt.md`. Assess issues independently — fix medium/high-risk issues (missing sections, contradictions, ambiguity that would cause wrong implementation); skip low-risk/stylistic issues. Stop the loop once no medium/high-risk issues remain. If medium/high-risk issues persist after 3 iterations, surface to the user — do not loop again.

## Completion Criteria

- [ ] All invariants were followed (one question per message, 2-3 approaches, spec committed).
- [ ] The user approved the committed spec, not just the verbal design discussed in conversation.
- [ ] Approaches were compared against dimensions chosen for this specific decision, not a generic checklist.
- [ ] If the design depends on external libraries or APIs, feasibility was verified through research or a spike.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the design is real.

Did I ignore any failure signals because the design now feels settled?

Am I exiting because the design is genuinely complete and approved, or because the structure now looks finished enough?

## Failure Signals

Stop and reassess if:
- You are discussing implementation details (library choices, file organization, syntax) before understanding the problem
- You are about to write the spec but major design sections are still unresolved or unapproved
- You proposed 2–3 approaches but they are variants of the same idea rather than genuinely different directions
- You have asked more than 5–6 questions without reaching approach proposals (see the "Enough questions" decision signal)
- The user pushes back on doing a design and wants to go straight to code — hold the HARD-GATE; do not compromise, explain why the design step matters
- You committed to an approach that depends on unverified third-party behavior — spike first
- You are comparing approaches but each is described in a different frame, making comparison impossible — use consistent dimensions
- You are designing around an external dependency without checking whether it actually supports the required capability
- You recommended a higher-complexity or higher-dependency approach when a simpler equivalent would meet all stated constraints
