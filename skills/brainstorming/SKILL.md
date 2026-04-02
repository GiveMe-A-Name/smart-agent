---
name: brainstorming
description: "Invoke when a feature, product change, workflow, or new component is being considered and no committed spec exists. Cost of unnecessary invocation: a short design conversation. Cost of missing: building the wrong thing — wrong interfaces, missing constraints, structure that must be undone. When uncertain whether a spec exists or is complete, invoke."
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

## Overview

Start by exploring the project context and sizing the scope. Through one question at a time, build enough understanding to propose 2–3 approaches. Once an approach is chosen, present the design section by section and get approval at each. Write and commit the spec, run it through review, and wait for the user to sign off before transitioning to implementation planning.

The judgment calls for each phase live in Decision Signals below.

## Decision Signals

**Scope flagging:** If the request describes multiple independent subsystems, flag decomposition immediately. Do not ask detail questions about a project that first needs to be broken apart. Help the user identify the independent pieces, their order, and then brainstorm only the first sub-project through the normal flow.

**Question ordering:** Start with purpose and constraints — what problem, for whom, with what limits. Move to success criteria and edge cases only after the problem is clear. Ask about technical preferences only when they would constrain the design direction, not as an opener. Starting with "what technology do you want to use?" before understanding the problem is a failure.

**Project context:** Explore files, docs, and recent commits before asking questions. Questions you could answer by reading the codebase waste the stakeholder's time. Follow existing patterns. Where existing code has problems that affect the work (overgrown files, tangled responsibilities), include targeted improvements as part of the design — not as unrelated refactoring.

**Visual companion:** Offer when upcoming questions will involve mockups, layouts, or diagrams. Do not offer for text-based choices. Even after the user accepts, decide per-question whether the content is genuinely visual or can be handled in the terminal. When offering, use this message — nothing else alongside it:

> "Some of what we're working on might be easier to explain if I can show it to you in a web browser. I can put together mockups, diagrams, comparisons, and other visuals as we go. This feature is still new and can be token-intensive. Want to try it? (Requires opening a local URL)"

Wait for the user's response before continuing. If they decline, proceed with text-only brainstorming. If they accept, read the detailed guide before proceeding: `skills/brainstorming/references/visual-companion.md`.

**Technical feasibility spike:** Sometimes a design question cannot be answered by discussion alone — it requires writing exploratory code. Recognize when to pause design and spike:
- An approach depends on a third-party library or API whose behavior is undocumented or ambiguous — write a minimal test to verify the capability before committing the design to it
- The performance characteristics of an approach are central to the design decision and cannot be estimated from first principles — measure with a prototype
- The integration between two systems is poorly understood — build the thinnest possible end-to-end connection to validate the seam

A spike is not implementation — it is design validation through code. Set a strict timebox (30 minutes to 2 hours). The spike answers one question. If the spike invalidates the approach, that's a success — you avoided building the wrong thing.

**External research integration:** Design decisions often depend on the capabilities and limitations of third-party libraries, services, or APIs. When evaluating approaches that rely on external dependencies:
- Verify the dependency actually supports the required capability — do not assume from the name or README alone
- Check for known limitations, breaking changes, or deprecation signals
- Consider the maintenance health of the dependency (last release, issue activity, bus factor)
- If the evaluation requires sustained research across multiple sources, that research work belongs to a separate phase — pause design, gather evidence, then return with facts

**Approach recommendation:** Prefer approaches that meet constraints with minimal dependencies. When options are equivalent in outcome, prefer the simpler one. Lead with your recommendation and reasoning — don't just list options and make the user choose without guidance.

**Structured approach comparison:** When presenting 2-3 approaches, compare them against consistent dimensions so the user can evaluate tradeoffs — not just pick a favorite. The judgment is in choosing which dimensions matter for this specific decision:
- If the approaches differ primarily in implementation effort, compare complexity and maintenance burden
- If the approaches differ in risk profile, compare what can go wrong, how likely, and how recoverable
- If the approaches have different dependency footprints, compare the external coordination and supply-chain implications
- If reversibility varies between approaches, make that explicit — a decision that is cheap to undo deserves less analysis than one that is permanent

Present as a table when comparing 3+ approaches across 3+ dimensions — the structure prevents approaches from being described in incompatible frames. But a two-sentence comparison of two approaches that differ in one dimension does not need a table.

**Enough questions:** Move to approach proposals when you can answer: what is it for, who uses it, what are the constraints, and what does success look like? If you cannot answer all four after 5–6 questions, move to proposals anyway — missing information is better surfaced through option trade-offs than through more questions.

**Design sufficiency:** The design is complete when you can answer: (1) what units exist and what are their interfaces, (2) how data flows through the system, (3) what error cases exist, (4) how each unit is tested. Any gap means the design needs more work. For narrow, well-understood requests, a brief bullet-point summary covering these four dimensions is sufficient. Write the spec once the user has approved each major section — do not write to discover gaps.

**Design isolation:** Break the system into units each with one clear purpose, well-defined interfaces, and independent testability. For each unit, answer: what does it do, how do you use it, what does it depend on? A unit you cannot understand or test independently needs its boundaries refined.

**Spec review judgment:** Dispatch a reviewer subagent using the prompt in `skills/brainstorming/templates/spec-document-reviewer-prompt.md`. The reviewer checks for: completeness (no TODOs or TBDs), internal consistency, clarity (no ambiguity that would cause the wrong thing to be built), appropriate scope (not covering multiple independent subsystems), and YAGNI (no unrequested features). Do not accept reviewer feedback wholesale. Assess each issue independently:
- **Medium/high risk** — missing sections, internal contradictions, ambiguity that would cause someone to build the wrong thing → fix and re-dispatch
- **Low risk / stylistic** — wording preferences, minor incompleteness, "could be more detailed" → skip or note as advisory

Stop the loop once no medium/high-risk issues remain, even if minor issues persist. If medium/high-risk issues are still unresolved after 3 iterations, surface to the user with a summary rather than looping again.

## Invariants

- One question per message — no exceptions
- Visual companion offer is its own message — no other content
- Always propose 2–3 approaches before presenting a design — unless the user has explicitly named an approach and said they do not need alternatives
- Every project gets a design — simple ones get a short one, not no design
- Spec is committed to git before the user reviews it
- Verbal approval of a design in conversation is not the same as approval of the written spec
- No unrelated refactoring — only improvements that directly serve the current goal

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
- You are asking multiple questions in a single message
- You are about to write the spec but major design sections are still unresolved or unapproved
- You proposed 2–3 approaches but they are variants of the same idea rather than genuinely different directions
- You have asked more than 5–6 questions without reaching approach proposals (see the "Enough questions" decision signal)
- The spec review loop has run 3 times without clearing medium/high-risk issues — surface to human, do not loop again
- The user pushes back on doing a design and wants to go straight to code — hold the HARD-GATE; do not compromise, explain why the design step matters
- You committed to an approach that depends on unverified third-party behavior — spike first
- You are comparing approaches but each is described in a different frame, making comparison impossible — use consistent dimensions
- You are designing around an external dependency without checking whether it actually supports the required capability

