---
name: technical-communication
description: "Write technical artifacts around a clear point of view and explain that point through simple concrete examples. Use when creating or revising design docs, RFCs, ADRs, postmortems, runbooks, READMEs, API docs, PR descriptions, commit messages, or other durable technical prose."
---

# Technical Communication

Give the document something worth saying, then make it easy to understand. A document succeeds when the reader can state its central point and retell the example that made the point clear.

## Principles

- **Form the point before writing the document.** Reduce the central issue to one sentence that states what the reader should understand, believe, choose, or do. The sentence must make a judgment, explain a mechanism, or establish a priority; a list of considerations is not a point of view [because structure without a governing idea produces complete-looking prose with no memorable content].
- **Take the strongest defensible position.** Preserve the user's position when one is given. Otherwise, use the available context to choose the interpretation or recommendation that best explains the issue, and state any assumption that materially shapes it. Do not retreat into an even-handed survey unless the user asked for one [because unranked options transfer the writing's central judgment back to the reader].
- **Explain the point with the smallest useful example.** Choose a familiar, concrete situation containing only the actors, action, and consequence needed to expose the mechanism [because every extra entity or detail competes with the idea the example is meant to clarify].
- **Prefer concrete language over abstract compression.** Name what changes, who encounters it, and what happens next. Introduce a technical term only when the reader needs it, then connect it to something observable.
- **Make every part serve the point.** Use the pattern `point -> example -> meaning`. Add qualifications, alternatives, facts, or implementation detail only when they sharpen, bound, or apply the central point.
- **Prefer a complete idea over a complete-looking document.** For explanatory or recommendation prose, default to two paragraphs: state the point and its priority, then give one example and explain its meaning. Add at most one further paragraph when a limitation would materially change how the reader applies the point [because covering every adjacent concern hides the judgment the document exists to communicate].
- **Keep evidence subordinate to understanding.** Do not lead with citations, benchmark tables, or an evidence inventory. Include evidence briefly when the user asks for it or when the reader must verify a consequential factual claim; place it after the point and example [because readers first need to understand the claim before deciding whether to inspect its support].

## Constraints

- State the central point in the opening paragraph. Do not begin with history, terminology, or a neutral catalogue of options.
- Do not present alternatives without saying which one the document favors, what priority drives that choice, or why no choice is currently possible.
- When a load-bearing explanation depends on an abstraction such as "decoupling", "scalability", "flexibility", or "maintainability", give a concrete case before moving to the next claim.
- Keep each example focused on one mechanism. Remove names, components, numbers, and edge cases that do not change what the example teaches.
- After an example, state its meaning in plain language. Do not make the reader infer why the example was included.
- Do not use an example as proof that a general claim is universally true, and do not invent facts to make a position sound authoritative.
- Give user-provided facts more weight than generic best practices or hypothetical causes. Do not invent a possible cause or alternative solution merely to make the position sound balanced.
- Once the point, example, and meaning answer the request, stop. Do not append an implementation plan, catalogue of alternatives, edge-case inventory, or best-practice section unless the user asked for it or its absence would materially misstate the point.
- Do not introduce a new component, process, or technical term when removing it would leave the central point and its causal explanation intact.
- Delete sections and sentences that neither state the point, explain it, show it, bound it, nor help the reader act on it.

## Calibration Examples

Weak — balanced but empty:

> Microservices and monoliths both have advantages and disadvantages. Microservices improve scalability and independent deployment, while monoliths can be simpler to operate. Teams should choose according to their circumstances.

Strong — point, simple example, meaning:

> A five-person team that has not yet validated its product should usually keep a monolith. Its scarce resource is iteration speed, not independent deployment. If changing one customer field requires coordinating three services and three deployments, service boundaries are already costing more than they return. Split a service only when a real scaling or ownership constraint appears.

Weak — terminology without an observable mechanism:

> Separating the notification module improves decoupling, maintainability, and system evolution.

Strong — concrete mechanism first:

> Notification changes should not require redeploying payment code. Today, changing an email template ships the payment service again; separating notification delivery removes that unnecessary release risk. That is the useful meaning of decoupling here.
