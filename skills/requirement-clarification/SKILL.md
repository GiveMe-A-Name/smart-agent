---
name: requirement-clarification
description: Use when a request is underspecified, ambiguous, or uses unclear terminology and the agent needs to clarify intent and unresolved human decisions before planning or implementation.
---

# Requirement Clarification

Clarify the task before planning or implementation.

Use this skill to turn a vague request into a clearer working understanding by
combining three inputs:

- the user's stated request
- repository evidence
- external knowledge when stable domain context matters

This skill is about better clarification behavior, not about filling out a
template. Its main job is to reduce ambiguity and compress the remaining
questions down to the few decisions that actually require the human.

## Boundary

This skill owns:
- turning vague or underspecified requests into clear task understanding
- identifying missing assumptions, hidden constraints, and conflicting interpretations
- deciding which unknowns can be answered from repository evidence
- resolving shallow repository unknowns directly when one or two targeted lookups are enough
- deciding which unknowns require external knowledge
- deciding which questions must be escalated to the human
- separating confirmed requirements from inferred context and unresolved decisions

This skill does not own:
- deep repository exploration methodology itself
- final code ownership analysis
- implementation planning or task sequencing
- candidate change point selection as a final deliverable
- verification strategy
- completion judgment

## Constitution

- Clarify before planning.
- Prefer the smallest evidence-gathering move that resolves the ambiguity.
- Resolve repository and stable external unknowns before asking the human.
- Distinguish user-stated goals, repository facts, external knowledge, and unresolved unknowns.
- Do not treat repository facts or external best practice as confirmed product intent.
- Escalate only irreducible product, business, or preference decisions to the human.
- Surface conflicting interpretations instead of silently choosing one.
- Produce clarified understanding, not generic exploration or a change plan.

## Clarification Heuristics

Use the smallest move that reduces ambiguity.

- First identify whether the unknown is about user intent, repository fact, or stable outside knowledge.
- Resolve local repository unknowns directly when one or two targeted lookups are enough; if deeper repository analysis is needed, treat it as supporting input rather than the result.
- Use external knowledge only when it changes how the request should be interpreted.
- Keep turning evidence into requirement language, and ask the human only about decisions that evidence cannot settle.
- Stop once the remaining uncertainty is small, explicit, and human-owned.

## Enough To Proceed

Clarification is sufficient when the request can be restated in concrete working terms, the main constraints are labeled by evidence strength, and the remaining uncertainty has been compressed into a small set of genuinely human decisions.

Do not continue clarifying just to gather more context.
Do not proceed if key intent is still being guessed silently.

## Self-Correction Signals

Stop and revise when:

| Signal | What it means | What to do |
|---|---|---|
| You are asking the human a question that repository evidence or stable external knowledge could likely answer | You are offloading avoidable ambiguity | Do the lookup or targeted research first; keep it shallow unless deeper analysis is truly required |
| You are tracing behavior across several files, trying to prove ownership, or producing a codebase analysis report while still calling it clarification | The ambiguity is no longer local, or the work has crossed the skill boundary | State that deeper repository analysis is required, and fold any useful result back into clarified intent |
| You are treating current repository behavior or external best practice as a confirmed requirement without proof | Facts or guidance are being mistaken for intent | Separate repository evidence, outside guidance, and user goals |
| You are escalating too many low-value questions, or you keep exploring after the remaining human decisions are already clear | Clarification has become open-ended exploration | Synthesize, narrow the decision surface, and stop |
| You are stating inferred constraints as confirmed facts, or acting certain even though intent was never confirmed | Evidence labels are missing, or certainty is overstated | Relabel claims as confirmed, inferred, or unknown, and re-examine what still needs human intent |
| You are jumping to implementation planning, change points, or verification strategy | Clarification has drifted into another skill boundary | Return to ambiguity classification and stop at clarified understanding |

## Verification Expectation

A good result:
- restates the request in clearer working terms
- separates user intent, repository facts, external knowledge, and unresolved decisions
- uses repository evidence and external knowledge only when they reduce genuine ambiguity
- labels what is confirmed, inferred, and still unknown
- ends with a small set of real human decisions rather than a long list of exploratory questions
- does not drift into planning, change design, or completion judgment

If the result mainly asks broad questions the agent could have answered itself, or if it confuses evidence with intent, the clarification is weak.
