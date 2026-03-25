---
name: requirement-clarification
description: Use when the user's request is ambiguous, underspecified, or internally conflicting and the agent needs to clarify what the user actually wants before planning or implementation. Do not use when the main missing step is code exploration or implementation.
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

## Trigger Logic

When the request is still ambiguous, identify what kind of unknown is actually blocking safe progress:

- `user intent`: what the user actually wants — only the user can resolve this
- `repository fact`: what the current code does, how it is structured, what patterns exist — can often be resolved with a targeted lookup
- `stable external knowledge`: well-established domain conventions or best practices — can often be resolved without asking the user

Use this skill when:
- the request is ambiguous, underspecified, or internally conflicting and proceeding risks working toward the wrong goal
- the main missing step is clarifying what the user actually wants, which constraints matter, or which tradeoff they intend
- some unknowns may be answered from shallow repository evidence or stable external knowledge, but the remaining ambiguity is still mainly about intent
- the agent needs to separate user-owned decisions from questions it can resolve directly from evidence

Do not use this skill when:
- intent is clear enough to support planning or implementation directly
- the main missing step is deep repository exploration, code ownership analysis, or implementation choice, not intent clarification
- the task is fully mechanical or already unambiguous
- the agent is using clarification as a default first step before every task

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

## Clarification Heuristics

- For each unknown, decide before asking: can a targeted lookup or stable external knowledge resolve this, or does it require a human decision? Only escalate what cannot be answered from evidence.
- Resolve local repository unknowns directly when one or two targeted lookups are enough; if deeper repository analysis is needed, treat it as supporting input rather than the result.
- Use external knowledge only when it changes how the request should be interpreted.
- When you must ask after exploration yields nothing, ask for a pointer to the code or location — something that lets you find the answer yourself. Asking the user to describe the problem shifts diagnosis to them and is a weaker fallback.
- When stated requirements explicitly conflict, the clarification question is about priority or tradeoff — not about what each requirement means individually. Name the conflict and ask the user to choose.
- Stop once the remaining uncertainty is small, explicit, and human-owned.

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

A good result restates the request in clearer working terms and ends with a small set of real human decisions, not a long list of exploratory questions.

Stop when these conditions are met; do not continue clarifying to gather more context.
Do not proceed if key intent is still being guessed silently.

If the result mainly asks broad questions the agent could have answered itself, or if it confuses evidence with intent, the clarification is weak.

See `examples/resolve-vs-escalate.md` for a contrast pair showing when a question should be answered from evidence versus escalated to the user.

See `examples/compress-to-root-question.md` for a contrast pair showing how to identify the single root question when multiple unknowns share the same underlying gap.

See `examples/conflicting-requirements.md` for a contrast pair showing how to handle requirements that explicitly contradict each other.
