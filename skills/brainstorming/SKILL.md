---
name: brainstorming
description: "Explore an open idea or problem to surface options, assumptions, and decision factors — let the user's thinking become clearer. TRIGGER when: user opens a topic for collaborative exploration, describes a goal without naming a solution, asks 'how should we approach X' or 'should we do X or Y', or presents alternatives without choosing."
---

# Brainstorming

Use this skill to improve the user's decision state while the topic is still open. The output is clearer thinking: grounded problem framing, distinct options, surfaced assumptions, decision factors, and uncertainty that can be acted on.

## Principles

- **Hold the problem open before narrowing.** When shape, scope, tradeoff, or approach is still unresolved, surface alternatives, assumptions, and decision factors before reducing the conversation to one path [because premature narrowing turns unexplored assumptions into wrong interfaces, missing constraints, or implementation work that has to be undone].
- **Ground the need before evaluating solutions.** If the user's framing prescribes an action such as "refactor X", "switch to Y", or "add Z", restate the underlying need and named assumptions before comparing approaches [because solution-shaped framings can hide the actual problem the options must solve].
- **Put materially different options on the table.** Offer approaches that differ in design, scope, ownership, cost, reversibility, or risk; do not count cosmetic variations of one approach as options [because the user cannot reason about tradeoffs without real alternatives].
- **Make hidden assumptions explicit.** Treat absolute language, fixed tools, prior decisions, and unexplained constraints as assumptions to surface or test [because implicit assumptions silently remove valid option spaces].
- **Name the decision factors.** When two or more options exist, compare them on observable axes such as reversibility, blast radius, maintenance cost, time-to-first-value, coupling, stakeholder impact, and uncertainty [because explicit factors let the user match options to actual constraints instead of choosing by vibe].
- **Introduce perspectives outside the user's frame.** Add blind spots, affected stakeholders, failure modes, non-action options, long-term costs, or cross-domain analogies when the conversation is staying inside one frame [because the agent's value in brainstorming is expanding the option space, not just reorganizing what the user already said].
- **Scale the session to the topic.** A small open question still gets useful exploration, but the response should be proportionate: a short topic may need one reframing, two options, or one decisive question rather than a full workshop [because skipping exploration loses signal, while over-expanding a small topic creates noise].

## Constraints

- Ask at most one question per message.
- If the latest user request is a factual lookup, a direct clarification, or a code/action request with no open decision, do not brainstorm; answer or act on that request.
- If the approach and interfaces are already decided and the remaining work is execution, do not reopen exploration unless the user asks to revisit the decision.
- Do not choose a direction on the user's behalf unless the user asks for a recommendation.
- Do not turn brainstorming into spec writing, implementation planning, sub-project decomposition, library selection, file organization, or syntax discussion before the root problem and options are grounded.
- Do not continue brainstorming after the user signals convergence with language like "let's go with X", "let's plan this out", or "start implementing".
- Do not leave the current goal to propose unrelated refactors, adjacent projects, or interesting but non-serving directions.

## Output Shapes

Use only the output shapes that match the current conversation. Do not walk this list as a checklist.

**Reframed problem statement**
- Use when: the user's opening statement prescribes a solution ("refactor X", "switch to Y", "add Z") rather than stating a need or constraint.
- Produce: rewrite the framing with embedded assumptions named and the underlying need stated explicitly.
- Reason: without reframing, every option that follows may solve the wrong problem.

**Distinct options on the table**
- Use when: the user has named one approach without saying it is fixed, or has described a goal without naming an approach.
- Produce: two or more approaches that lead to materially different designs, not variants of one idea.
- Watch for: if all options share the same underlying assumption, keep diverging.

**Surfaced assumptions**
- Use when: the user uses absolute language ("we have to", "we can't", "obviously"), references a system/tool/constraint as fixed without justifying it, or treats a prior decision as non-negotiable.
- Produce: implicit constraints or beliefs as explicit assumptions the user can confirm, reject, or revise.
- Reason: assumptions left implicit silently exclude option spaces.

**Decision factors**
- Use when: two or more options are on the table and comparison has begun.
- Produce: the axes along which options differ, such as reversibility, blast radius, maintenance cost, time-to-first-value, coupling, uncertainty, or stakeholder impact.
- Reason: explicit factors let the user compare against constraints instead of choosing by gut feel.

**Open questions**
- Use when: comparison hits a question neither party can answer in the conversation.
- Produce: facts to gather, people to ask, experiments to run, or constraints to verify before choosing.
- Reason: making uncertainty explicit prevents premature commitment.

**Blind spots introduced**
- Use when: detect signals in `references/blind-spots.md` match the current conversation state.
- Produce: one relevant failure mode, frame lock, missing perspective, inversion, time horizon, or cross-domain analogy.
- Reason: without injected perspectives, the conversation may only reorganize what the user already thought.
