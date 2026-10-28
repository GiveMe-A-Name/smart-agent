---
name: technical-communication
description: "Write clear, accurate technical documents that serve their intended readers and purpose. Use when creating or revising technical launch announcements, design docs, RFCs, ADRs, postmortems, runbooks, READMEs, API docs, PR descriptions, commit messages, or other durable technical prose."
---

# Technical Communication

Help the user write effective technical documents: communicate from the appropriate author's perspective, give the intended readers the information they need, and enable the understanding, decision, or action the document is meant to support. Judge success by whether the document accomplishes that purpose. Clear viewpoints, concrete examples, and concise prose are techniques to use when they help achieve it.

## Establish the Writing Context

Before drafting, establish four things from the user's request and supplied material:

- **Artifact:** What kind of material is this, such as a launch announcement, design proposal, or tutorial? Its genre shapes what the reader expects it to deliver.
- **Author's perspective:** Who is speaking, in what role, and with what relationship to the reader? A framework maintainer introducing their framework speaks from a different position than an independent reviewer assessing it.
- **Reader:** Who is this for, what do they already know, and what do they care about? Framework users and framework contributors may need different explanations of the same change.
- **Purpose:** What should the reader understand, believe, decide, or do afterward? Make the intended outcome specific enough to guide what belongs in the document.

Use this context to choose the content, structure, depth, voice, and ways of explaining. Author perspective and audience must affect what the document says, not just its pronouns or tone. Infer what is clear from context; clarify missing information with the user when it would materially change the draft. Mention missing information in the finished document only when it affects the reader's understanding or use. These are drafting decisions, not a mandatory preamble in the finished document. Preserve them when revising unless the user changes the brief.

## Principles

- **Let the purpose govern what belongs.** Identify the questions this reader needs answered for the document to achieve its purpose. Select and prioritize material accordingly. A framework launch announcement should connect capabilities to users' needs; an internal design document may need the implementation rationale behind those same capabilities.
- **Organize for how the reader will use the document.** Make the subject and relevance clear early, then order the material to support the reader's task. A proposal may lead with a recommendation, a tutorial may follow the sequence of work, and a reference may organize information for lookup. Respect the requested format and use enough context for the reader to follow.
- **Make recommendations clear when the document calls for them.** Preserve the user's position when one is given. For recommendations, state what you recommend and explain why it fits the reader's situation. For descriptive or reference material, make the facts and relationships clear. For a requested neutral comparison, present the options using criteria relevant to the reader.
- **Choose explanations that help this reader understand.** Prefer concrete language: name what changes, who encounters it, and what happens next. When an abstraction hides an important mechanism, a small, familiar example can make it visible. The pattern `point -> example -> meaning` is useful for such explanations; use it where it helps, without making it the structure of every document. A direct explanation, definition, or table may already be sufficient.
- **Be concise while meeting the reader's needs.** Remove repetition and irrelevant detail. Keep the context, qualifications, and instructions necessary for the intended use, even when they require more space. Set length by the purpose and requested scope; do not impose a fixed paragraph count or stop merely because one point has been explained.

## Constraints

- When using an example, keep it focused on what it needs to explain and make its relevance clear. Remove details that distract from that purpose.
- Stay faithful to the supplied material; do not invent facts.
- Build on the user's concrete content. Avoid generic best-practice filler and unrelated discussion.
- Stay within the requested scope. Add sections or technical terms when they serve the reader's task; remove material that contributes neither necessary information nor useful explanation. Finish when the document fulfills its intended purpose.

## Calibration Examples

These excerpts illustrate choices for particular readers and purposes; they are not templates for every document.

Scenario — a launch announcement from framework maintainers to application developers, intended to encourage a first trial. In this hypothetical brief, the supplied facts are a shared retry module replacing duplicated internal implementations, built-in retries for transient request failures, and a runnable quickstart.

Weak — centered on the maintainers' implementation work:

> We consolidated retry logic into a shared module, removing duplicate implementations across the framework. This makes future retry changes easier for us to maintain.

Strong — maintainer perspective, user value, and a next step:

> Our new framework handles transient request failures with built-in retries. When a request fails temporarily, the framework retries it for you, saving you from writing that retry loop in your application. Try the quickstart to make your first request.

The announcement connects the maintainers' work to a user's task and gives the reader a way to try it. An internal design note about the same module would instead explain the implementation choices that contributors need to understand.

Scenario — a recommendation for a five-person team that has not yet validated its product.

Weak — leaves the requested judgment to the reader:

> Microservices and monoliths both have advantages and disadvantages. Microservices improve scalability and independent deployment, while monoliths can be simpler to operate. Teams should choose according to their circumstances.

Strong — point, simple example, meaning:

> A five-person team that has not yet validated its product should usually keep a monolith. Its scarce resource is iteration speed, not independent deployment. If changing one customer field requires coordinating three services and three deployments, service boundaries are already costing more than they return. Split a service only when a real scaling or ownership constraint appears.

Weak — terminology without an observable mechanism:

> Separating the notification module improves decoupling, maintainability, and system evolution.

Strong — concrete mechanism first:

> Notification changes should not require redeploying payment code. Today, changing an email template ships the payment service again; separating notification delivery removes that unnecessary release risk. That is the useful meaning of decoupling here.

Scenario — an API reference for developers checking a timeout parameter. Assume the supplied contract defines `timeout_ms` as a positive integer in milliseconds, defaults it to `5000`, and raises `TimeoutError` when the request exceeds it.

Effective — directly supplies the information needed for lookup:

> `timeout_ms`: positive integer, in milliseconds; default `5000`. If the request exceeds this duration, it raises `TimeoutError`.

The reference fulfills its purpose through a precise definition. It needs neither a recommendation nor an illustrative story.
