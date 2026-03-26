---
name: understanding-codebases
description: Use when proceeding without reading the code would require guessing about structure, ownership, or behavior. Invoke by default — issue descriptions, PR text, and user explanations are hypotheses, not code evidence.
---

# Codebase Understanding

Build grounded understanding of the code before suggesting implementation — focused on the slice relevant to the current question, not a tour of the whole repository.

Use this skill to identify the files and modules that matter, trace how behavior flows through the relevant code, and explain where a change likely belongs. Read the problem-relevant context thoroughly; expand only when a new hypothesis requires it; stop once the answer no longer requires intuition. This skill analyzes code structure; it does not clarify product requirements, create an implementation plan, or decide whether work is complete.

## Trigger Logic

**Invocation default**: The cost of an unnecessary invocation is minor overhead. The cost of a missed invocation is unfounded conclusions that contaminate all subsequent work. When the task is not confirmed — from already-seen code evidence, not impression or prior knowledge — to involve only a single self-contained file with no cross-file tracing needed, invoke this skill.

**What counts as code evidence**: reading actual source files and tracing behavior. Issue descriptions, PR summaries, bug reports, and user explanations are hypotheses about where the problem is — not evidence of what the code does. A hypothesis that "the problem is in module X" does not confirm scope; it names where to look first.

Use this skill when:
- the user is asking how part of the codebase works, where behavior is implemented, or where a change should go
- answering safely requires tracing behavior across multiple files or layers rather than naming one plausible file
- the question is concrete, but current repository evidence is still too thin to answer confidently
- a single obvious file or one or two targeted lookups do not ground the answer well enough
- tracing call flow, comparing nearby patterns, or proving ownership matters before suggesting where work belongs
- proceeding would require guessing about code structure, ownership, or behavior


"Two targeted lookups" means at most two tool calls tied to the current hypothesis set — not one lookup per likely location, not repeated checks that only reconfirm the same absence, and not curiosity-driven browsing. If the first lookup already suggests the repo lacks the evidence needed to answer the current premise safely, stop and self-correct rather than broadening the search.

Before deepening repository exploration, distinguish among these cases:

- `intent ambiguity`: the request mainly needs clarification or human decisions; repository exploration does not replace that, but it often informs what questions to ask — if codebase evidence is thin, explore first, then clarify
- `repo-fact gap`: the question is concrete, but repository evidence is missing; use this skill only if limited targeted lookups still do not ground the answer
- `already answerable`: you have already run targeted lookups that produced concrete, sufficient code evidence — not intuition or prior knowledge about the codebase; do not invoke this skill

## Boundary

This skill owns:
- building a task-specific repository slice for the current question
- tracing enough call flow to ground ownership, behavior, or candidate change points
- comparing nearby implementations when local patterns matter
- stating what is proven, hypothesized, and still unknown

This skill does not own:
- clarifying product intent, scope, or acceptance criteria
- writing a change plan or sequencing implementation work
- final engineering judgment about how a change should be implemented once repository evidence is understood
- recommending verification strategy or completion judgment

## Core Moves

These are understanding moves, not a fixed sequence. Use the ones that help answer the current code-understanding question.

- Anchor on the current question.
- Identify the current hypothesis before spending lookup budget.
- Build the smallest repository slice that can answer it safely.
- Trace toward behavior-changing code instead of stopping at facades.
- Compare nearby patterns when they materially inform the answer.
- Use a second lookup only when it tests a materially different hypothesis, such as "the first entry-path guess was wrong," rather than repeating the same absence check in another likely place.
- Stop once the answer is grounded well enough to avoid intuition.

## Grounded Understanding

You have enough understanding for the current question when, from code evidence, you can do the relevant subset of the following:

- explain the repository slice that matters to this question
- trace enough call flow to answer safely
- point to concrete evidence for ownership, behavior, conventions, or candidate change points when relevant
- state the major unknowns that remain

If you still need intuition to bridge the answer, keep investigating.

## Self-Correction Signals

Stop and revise when:

- you are suggesting changes before building a local repository slice
- you stopped at export, registration, config, or facade layers
- you listed files without explaining relationships or call flow
- you are treating guesses about ownership or change placement as facts
- a negative lookup already supports "missing repo evidence," but you keep searching without a materially different hypothesis
- your follow-up lookup changes paths or tools but only reconfirms the same absence
- you are drifting into planning or still exploring after the question is already grounded

## Judgment in Practice

**Task**: "Where does the auth token get validated?"

**Decision 1 — Should I invoke this skill?**
The question is concrete, but answering it safely requires more than naming a plausible file. I need to trace from request entry toward behavior-changing code and distinguish orchestrators from the real validation point. → Invoke this skill.

**Decision 2 — What is my starting hypothesis?**
My first hypothesis is that auth validation starts in request middleware. That is only a starting point, not the answer. I need repository evidence.

**Decision 3 — First lookup: entry path found. Is that enough?**
I found `middleware/auth.ts` with a `validateToken` call. This grounds the entry path, but the file appears to delegate the real work. That is useful evidence, not yet the behavior-changing code. → Continue.

**Decision 4 — Second lookup: follow delegation, not curiosity.**
My next hypothesis is that the delegated module contains the actual validation logic or leads directly to it. I spend the second lookup on the module called by `middleware/auth.ts`, such as `services/token.ts`, because it tests a new hypothesis rather than repeating the same absence check elsewhere.

**Decision 5 — What counts as enough evidence?**
If `services/token.ts` performs signature checks, expiry checks, or calls a concrete verifier that I actually inspected, I can now point to the real validation path and stop. If it only delegates again and I have not traced into the next layer, I should not present that downstream verifier as proven. I should say the validation path likely continues there, but that final step remains unverified.

**Decision 6 — How do I report the result?**
State the role of each file precisely. For example: `middleware/auth.ts` is the request entry and orchestration layer; `services/token.ts` contains or routes the validation flow; any downstream verifier is only a proven validation point if I actually traced into it. Anything not directly confirmed from code stays labeled as a hypothesis or unknown.

**What this skill prevents here:**
- Stopping at the first plausible file and calling it the answer
- Treating a function name like `validateToken` as proof of where validation really happens
- Spending more lookups just to reconfirm the same absence in neighboring files
- Presenting an untraced downstream module as a proven change point
- Drifting into planning after the repository slice is already grounded

See `examples/fake-new-hypothesis.md` for a contrast pair showing how a lookup that looks like a new hypothesis can actually just be rechecking the same absence.
