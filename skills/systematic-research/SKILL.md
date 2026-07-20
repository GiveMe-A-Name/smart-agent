---
name: systematic-research
description: "Research external topics and questions into clear, well-scoped, evidence-grounded mental models and explanatory reports. Use when the user asks to investigate, understand, compare, or get up to speed on a technology, concept, ecosystem, practice, or current external subject. Do not use when the source of truth is local code, tests, or logs, or when the task only asks to rewrite material already provided."
---

# Systematic Research

Research succeeds when the reader can explain the topic, see how its important parts relate, and judge what matters. A long source list or a large number of searches is not the outcome.

## Research Shapes

Adapt the depth and output to the request. Research breadth and report depth are separate decisions: the agent may need a broad internal map to avoid a misleading answer, but the user should see only the detail needed for the requested outcome.

- **Topic orientation** — The user names a subject such as “Research React Server Components.” Default to a concise orientation that lets a technically literate reader understand the central model and practical significance quickly. Do not interpret the word “research” by itself as a request for a comprehensive report.
- **Targeted question** — The user asks whether a current version supports a behavior or what an external system does. Answer the exact question with version-appropriate evidence; do not expand it into a topic survey unless the surrounding concepts are necessary to avoid a misleading answer.
- **Comparison or decision** — The user asks which option to use or how approaches differ. Establish decision criteria from the user’s goal, research the differences that change the choice, and give a recommendation or explain why the evidence does not support one.
- **Deep research** — Expand the full topic map only when the user explicitly asks for an in-depth, comprehensive, systematic, literature-style, or long-form investigation, or when an agreed deliverable requires that depth. Preserve prioritization even in a deep report; depth is not permission to include every fact found.

## Core Method

### 1. Frame the research

State the question the report must answer and set a useful boundary. Infer reasonable defaults from the request instead of making the user design the research plan.

For a bare topic, identify:

- what a reader should understand or be able to decide afterward;
- which audience and level of detail are reasonable;
- which neighboring subjects must be distinguished but not researched in full;
- which parts are stable concepts and which depend on current versions, products, or ecosystem state.

Unless the user signals otherwise, frame the deliverable as a short, decision-ready orientation rather than a reference document or implementation plan.

Ask for clarification only when different interpretations would produce materially different reports. Keep the working frame in research notes; do not make the user approve an internal checklist.

### 2. Map the topic

Build an initial knowledge map before collecting details. The map should contain concepts and relationships, not just section names. Refine it as evidence changes the understanding, but do not let the first source define the report outline.

Choose the perspectives that make this topic understandable. Common lenses include:

- definition, boundary, and purpose;
- origin or problem context;
- core mechanism, participants, lifecycle, or data flow;
- relationships to adjacent or commonly confused concepts;
- practical use, ecosystem, adoption, and maturity;
- benefits, constraints, tradeoffs, and failure modes;
- appropriate and inappropriate use cases;
- current disagreements, evolution, and open questions.

These are prompts for internal coverage, not mandatory report headings. Add topic-specific lenses and omit generic ones that do not improve the reader’s model. Mark the map’s load-bearing nodes: claims or relationships that would change the explanation, comparison, or recommendation if wrong. A broad map protects against omissions; it does not require every branch to appear in the output.

### 3. Investigate the map

Research to fill important gaps and test the load-bearing parts of the map. In the default mode, stop after the central model and consequential implications are supported; do not deepen peripheral branches merely because they are interesting.

- Use primary or authoritative sources for definitions, specifications, contracts, versions, and official behavior.
- Use independent implementation evidence, maintainer discussion, credible practitioner experience, or executable checks for operational limits, adoption, failure modes, and behavior that documentation does not settle.
- Trace consequential claims in secondary sources to their origin. Search snippets and AI-generated summaries are discovery aids, not evidence.
- Check dates, versions, environments, and evaluation settings before applying a finding to the requested scope.
- Seek contrary evidence when a claim is disputed, surprising, recommendation-changing, or supported only by an interested party. Do not perform a ceremonial disconfirmation search for every ordinary fact.
- When sources conflict, explain whether the difference comes from version, scope, method, or unresolved disagreement. Do not silently choose the convenient source.
- When documentation cannot settle behavior, prefer a small reproducible test over further ambiguous searching.

Treat unsupported content as an inference, disputed claim, or unknown. Formal source tiers and confidence labels are optional; use them only when they help the reader judge consequential uncertainty.

### 4. Synthesize for understanding

Organize the report in the order a reader should learn the topic, not the order in which sources were found.

Lead with the central point: what the subject is, why it exists, and what matters most about it. Then establish the smallest accurate mental model. Use one concrete example, request path, lifecycle, or scenario to explain the main mechanism, and state what the example teaches.

### Default depth

For an ordinary topic request, produce a briefing that can be read in roughly five minutes. Prefer:

- the central conclusion and why the topic matters;
- the smallest useful model of how it works;
- only the three to five implications, distinctions, or tradeoffs most important to understanding or judgment;
- one current caveat or unresolved issue when it materially changes the picture;
- direct sources for consequential claims.

Do not add a migration plan, production checklist, exhaustive history, full option landscape, capacity test matrix, or catalogue of edge cases unless the user asks for it or its absence would make the conclusion unsafe or misleading.

### Deep-research depth

When deeper research is explicitly requested, expand the parts of the knowledge map that serve that request. A deep report may cover history, mechanisms, ecosystem, competing views, operational concerns, and open questions, but it still needs an executive summary and clear prioritization so the reader can stop after forming the main judgment.

These are output contracts, not fixed tables of contents. Merge, reorder, or omit sections to preserve a clear explanation. Keep evidence subordinate to understanding: cite claims where readers may need to verify them, but do not turn the report into a research log.

## Completion Standard

Before finishing, check the report from the reader’s perspective:

- Can the reader explain what the topic is, why it exists, and how its central mechanism works?
- Are the important concepts connected rather than presented as an inventory?
- Are boundaries and commonly confused neighboring concepts clear?
- Does the coverage include the tradeoffs, constraints, and practical consequences that materially shape understanding?
- Are current, disputed, behavioral, and recommendation-changing claims supported by appropriate evidence or marked unresolved?
- Does the opening give a useful overview before detail, and does each later section deepen that model?
- Did every expanded section earn its place from the user’s requested depth or from a fact that changes the main understanding?
- Would more searching change the knowledge map or a consequential conclusion? If not, stop.

Do not equate completeness with length. A focused report that creates an accurate, usable mental model is more complete than an exhaustive catalogue that leaves the reader to assemble the topic themselves.

## Boundaries

- Do not use external research to answer questions whose source of truth is the local repository, its tests, logs, or runtime state.
- Do not present one framework’s implementation as the universal definition of a broader concept.
- Do not hide a narrow research scope behind broad conclusions.
- Do not pad the report with history, option lists, benchmark tables, or citations that do not change understanding.
- Do not turn a default orientation into an implementation guide, operational runbook, or exhaustive reference because additional useful material was found.
- Do not expose chain-of-thought, raw search notes, or a source-by-source diary unless the user explicitly asks for research provenance.
