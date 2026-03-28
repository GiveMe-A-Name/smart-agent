---
name: requirement-clarification
description: Invoke when the request leaves intent, constraints, or tradeoffs unresolved that only the user can answer. When uncertain whether requirements are clear enough, invoke. Cost of unnecessary invocation: brief clarification round. Cost of missing: building toward the wrong goal.
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

**Invocation default**: Skipping clarification when intent is genuinely unclear converts hidden assumptions into wasted work — a plan or implementation built toward the wrong goal. The cost of an unnecessary clarification round is minor friction. The cost of missing it is delivering something the user did not want. Invoke this skill when open questions about goals, constraints, or tradeoffs would change the approach if answered differently.

Use this skill when:
- the request leaves intent, constraints, or tradeoffs unresolved in ways that would change the approach
- the request is ambiguous, underspecified, or internally conflicting and proceeding risks working toward the wrong goal
- some unknowns may be answered from evidence, but the remaining ambiguity is still about what the user actually wants
- the request mentions implementation ideas or code locations, but those details do not resolve the underlying goal question


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

## Invariants

- For each unknown, decide before asking: can evidence resolve this, or does it require human decision?
- Only escalate what cannot be answered from repository evidence or stable external knowledge
- Separate confirmed requirements from inferred context and unresolved decisions
- Stop once remaining uncertainty is small, explicit, and human-owned
- **Before exiting this skill, you MUST complete the Self-Check section at the end**

## Clarification Heuristics

- For each unknown, decide before asking: can a targeted lookup or stable external knowledge resolve this, or does it require a human decision? Only escalate what cannot be answered from evidence.
- Resolve local repository unknowns directly when one or two targeted lookups are enough; if deeper repository analysis is needed, build that understanding first and fold its findings into clarified intent — do not attempt deep code tracing inside this skill.
- Use external knowledge only when it changes how the request should be interpreted.
- When you must ask after exploration yields nothing, ask for a pointer to the code or location — something that lets you find the answer yourself. Asking the user to describe the problem shifts diagnosis to them and is a weaker fallback.
- When stated requirements explicitly conflict, the clarification question is about priority or tradeoff — not about what each requirement means individually. Name the conflict and ask the user to choose.
- Stop once the remaining uncertainty is small, explicit, and human-owned.

## Self-Correction Signals

Stop and revise when:

| Signal | What it means | What to do |
|---|---|---|
| You are asking the human a question that repository evidence or stable external knowledge could likely answer | You are offloading avoidable ambiguity | Do the lookup or targeted research first; keep it shallow unless deeper analysis is truly required |
| You are tracing behavior across several files, trying to prove ownership, or producing a codebase analysis report while still calling it clarification | The ambiguity is no longer local, or the work has crossed the skill boundary | Pause, build the codebase understanding needed, and fold its findings back into clarified intent |
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

## Self-Check Before Exiting

- [ ] Did I follow all invariants (resolve from evidence first, separate confirmed from inferred)?
- [ ] Did I catch any self-correction signals?
- [ ] Does the result restate the request with a small set of real human decisions?
- [ ] Am I exiting because requirements are genuinely clarified, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
