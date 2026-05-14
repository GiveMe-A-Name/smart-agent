---
name: brainstorming
description: "Explore an open idea or problem to surface options, assumptions, and decision factors — let the user's thinking become clearer. TRIGGER when: user opens a topic for collaborative exploration, describes a goal without naming a solution, asks 'how should we approach X' or 'should we do X or Y', or presents alternatives without choosing."
---

# Brainstorming

Explores an open idea through dialogue. The output is clearer thinking — surfaced options, assumptions, and decision factors. The user decides when to stop and what to do next.

## Trigger Signals

**Invocation default**: Skipping exploration when it is needed converts unresolved questions into implementation mistakes — wrong interfaces, missing constraints, or structure that has to be undone. The cost of an unnecessary exploration is a short conversation. The cost of missing it is building the wrong thing. Invoke this skill whenever the topic is open and at least one decision about shape, scope, tradeoff, or approach remains unexamined.

Invoke when: the topic is open and at least one decision about shape, scope, tradeoff, or approach remains unexamined.

Do not use this skill when:
- the approach and interfaces are already decided and the remaining step is writing code against that decision
- the question is a factual lookup or a clarification with no open decisions

## Capability Boundary

This skill covers divergent exploration: grounding the problem, surfacing options, making assumptions visible, naming decision factors, and introducing perspectives outside the user's frame.

This skill does NOT own:
- Choosing a direction on the user's behalf
- Spec writing or implementation planning
- Sub-project decomposition

The user owns convergence. When the user signals they are ready to stop exploring and move on (e.g. "let's go with X", "let's plan this out", "start implementing"), exit this skill — do not continue to diverge.

## Invariants

- One question per message — no exceptions
- Actively introduce perspectives, options, and questions from outside the user's original frame — divergence is the core motion of this skill
- Every exploration matters — short topics get a short session, not no session
- No unrelated refactoring — only directions that serve the current goal

## What Good Brainstorming Looks Like

A session is going well when one or more of these outputs are emerging. None is mandatory — produce the ones the user's situation calls for. The session is complete when the user says so, not when this list is filled.

**Reframed problem statement**
- *What it is*: the user's original framing rewritten with embedded assumptions named and the underlying need stated explicitly
- *Why it matters*: original framings often prescribe a solution shape that hides the real problem (e.g. "we need to refactor X" hides "X is hard to maintain"). Without reframing, every option that follows solves the wrong problem.
- *Produce when*: the user's opening statement is a verb-object that prescribes a solution ("refactor X", "switch to Y", "add Z") rather than a description of a need or constraint

**Distinct options on the table**
- *What it is*: two or more approaches placed side by side that lead to materially different designs — not variants of one idea
- *Why it matters*: a single option is not a choice. The user cannot reason about tradeoffs without alternatives to compare against.
- *Produce when*: the user has named one approach without stating they only want that one; or has described a goal without naming any approach
- *Watch for*: options that share the same underlying assumption are not distinct — keep diverging

**Surfaced assumptions**
- *What it is*: implicit constraints or beliefs the user was treating as fixed, made explicit so they can be questioned
- *Why it matters*: assumptions left implicit silently exclude entire option spaces. Making them visible expands what the user can consider — even if the user re-confirms the assumption afterward, the confirmation is now deliberate.
- *Produce when*: the user uses absolute language ("we have to", "we can't", "obviously"), references a system/tool/constraint as fixed without justifying it, or treats a prior decision as non-negotiable

**Decision factors**
- *What it is*: the axes along which options differ — e.g. reversibility, blast radius, maintenance cost, time-to-first-value, coupling
- *Why it matters*: without explicit factors, choosing between options collapses to gut feel. With them, the user can match options to their actual constraints, and can defer the choice on principled grounds.
- *Produce when*: two or more options are on the table and comparison talk has begun

**Open questions**
- *What it is*: what would need to be answered before a confident choice is possible — facts to gather, people to ask, experiments to run
- *Why it matters*: brainstorming exposes uncertainty. Making the uncertainty itself an output prevents premature commitment and gives the user a legitimate next step that is not "decide now".
- *Produce when*: a comparison hits a question neither party can answer in the conversation

**Blind spots introduced**
- *What it is*: perspectives the user is unlikely to reach alone — failure modes, frame locks, etc. (see `references/blind-spots.md`)
- *Why it matters*: the agent's asymmetric value in brainstorming is breaking the user's cognitive frame. Without injected perspectives, the conversation only reorganizes what the user already thought.
- *Produce when*: detect signals in `references/blind-spots.md` match the current conversation state

## Failure Signals

Stop and reassess if:
- You are discussing implementation details (library choices, file organization, syntax) before the root problem is grounded
- The "options" you have surfaced are variants of the same idea rather than genuinely different approaches — you are not actually diverging
- You are pushing the user toward a decision they have not asked to make — convergence is the user's call, not yours
