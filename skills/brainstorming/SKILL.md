---
name: brainstorming
description: "Explore an open idea or problem until a clear direction is agreed. TRIGGER when: user opens a topic for collaborative exploration, describes a goal without naming a solution, asks 'how should we approach X' or 'should we do X or Y', or presents alternatives without choosing."
---

# Brainstorming Ideas Into Designs

Explores design options through open dialogue until the user has confirmed a direction. The output is a clear, agreed design — not a spec, not code.

<HARD-GATE>
Do NOT write code, scaffold a project, or begin any implementation work until the user has explicitly confirmed the design direction in conversation.
</HARD-GATE>

## Trigger Signals

**Invocation default**: Skipping design when it is needed converts unresolved questions into implementation mistakes — wrong interfaces, missing constraints, or structure that has to be undone. The cost of an unnecessary design session is a short conversation. The cost of missing it is building the wrong thing. Invoke this skill whenever no committed spec exists and the work involves decisions about what to build or how it should work.

Invoke when: no committed spec or agreed design exists, and at least one decision about shape, scope, tradeoff, or approach remains open.

Do not use this skill when:
- a committed spec already exists, the approach and interfaces are decided, and the remaining step is writing code against that spec
- the question is a factual lookup or a clarification with no open design decisions

## Capability Boundary

This skill covers design exploration: understanding the problem, surfacing options, comparing tradeoffs, and reaching a confirmed direction.

This skill does NOT own:
- Spec writing or spec review
- Committing design documents to git
- Implementation planning or test design — out of scope for this skill even after design is confirmed
- Sub-project decomposition beyond identifying that it's needed
- Designing a scope the user hasn't yet agreed to decompose

## Invariants

- One question per message — no exceptions
- Always surface 2–3 genuinely different directions before converging — unless the user has explicitly named an approach and said they do not need alternatives
- Every design gets explored — simple ones get a short session, not no session
- No unrelated refactoring — only improvements that directly serve the current goal

## What Good Looks Like

Use this section to assess where the session is. These are not steps to execute — they are states to evaluate against.

**The problem is grounded**

The session is anchored to the root problem, not the surface framing. The user's original statement contains embedded assumptions — "we need to refactor the code" assumes structural change is the solution; the root problem is likely "code is hard to maintain." Grounded means those assumptions have been named and the underlying need is explicit.

Not grounded: the discussion is organized around the user's original description without questioning what it assumes.

**Divergence is genuine**

Divergence is happening when:
- Information has surfaced that was absent from the original prompt — new constraints, user needs, or technical realities the user hadn't stated
- The options on the table are fundamentally different, not variants of the same approach — they lead to materially different designs with different tradeoffs
- The user's cognitive frame has been actively challenged — perspectives they would not reach on their own have been introduced (see `references/blind-spots.md`)

Divergence is not happening when the agent is only asking clarifying questions within the user's original frame, or when all options cluster around the user's opening assumption.

**Convergence is real**

Convergence is real when:
- The chosen direction carries explicit reasoning — "because X, not Y" — not just a stated preference
- Rejected alternatives were consciously set aside with understood reasons, not quietly dropped
- The direction that emerged is traceable to the dialogue itself — not a position either party held at the start

Convergence is not real when the user agrees with the agent's recommendation without the reasoning being examined, or when the final direction is identical to the user's opening assumption.

## Completion Criteria

- [ ] The problem was grounded — root problem named, embedded assumptions surfaced
- [ ] Divergence was genuine — new information surfaced, fundamentally different directions explored, cognitive frame challenged
- [ ] Convergence is real — chosen direction has explicit reasoning, rejected alternatives understood, direction traceable to dialogue
- [ ] The user has explicitly confirmed the direction in conversation

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the apparent completeness of the design is real.

Did I ignore any failure signals because the design now feels settled?

Am I exiting because the design is genuinely complete and approved, or because the structure now looks finished enough?

## Failure Signals

Stop and reassess if:
- You are discussing implementation details (library choices, file organization, syntax) before the root problem is grounded
- Major design decisions are still unresolved but implementation work is about to begin
- You surfaced 2–3 directions but they are variants of the same idea rather than genuinely different approaches
- The user wants to go straight to code without confirming a direction — hold the HARD-GATE; explain why a confirmed direction matters before implementation
