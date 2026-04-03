---
name: systematic-research
description: "Invoke when the task requires information from outside the local codebase — documentation, APIs, library behavior, external systems, or current state of the world. Cost of unnecessary invocation: a short framing pass. Cost of missing: conclusions that sound authoritative but rest on unverified assumptions or stale knowledge. When in doubt, invoke."
---

# Systematic Research

Gather, evaluate, and synthesize external information with the rigor of an intelligence analyst and the epistemic humility of a scientist. The goal is grounded, cross-validated conclusions with calibrated confidence — not comprehensive coverage, not speed, and not confirming what you already believe.

Your internal knowledge is a starting point for questions, never a substitute for evidence. Treat it as an untested hypothesis until corroborated by external sources.

## Trigger Logic

**Invocation default**: Unstructured searching feels productive but produces anchored, confirmation-biased results. The cost of invoking unnecessarily is a short framing pass. The cost of skipping is conclusions that sound authoritative but rest on unverified assumptions, single sources, or stale knowledge. When external information is needed and the answer is not already supported by verified evidence in context, invoke.

External information means anything outside the local codebase: web search results, official documentation, API references, GitHub issues in external repos, RFCs, research papers, expert analyses, external system behavior.

## Capability Boundary

This skill owns:
- decomposing open-ended questions into verifiable sub-questions
- hypothesis-driven evidence gathering with active use of web search tools
- cross-validation of claims across independent sources
- source credibility assessment and provenance tracing
- synthesizing findings with confidence calibrated to evidence quality
- recognizing and counteracting cognitive biases during research

This skill does not own:
- deciding whether gathered information is sufficient for a final decision — that belongs to the skill or workflow consuming the research
- product or requirement decisions
- investigating the local codebase (that requires codebase understanding, not external research)

## Invariants

- Never issue a search with the original vague question — decompose into specific sub-questions first, each targeting a verifiable hypothesis.
- Cross-validate every load-bearing claim across independent sources before treating it as established. Sources that all trace back to the same original count as one, not many.
- Trace every secondary-source claim to its primary source when the conclusion depends on it.
- Absence of evidence is not evidence of absence — distinguish "I found evidence X is true" from "I found no evidence X is false."
- Use web search tools actively — internal knowledge cannot substitute for external verification of APIs, versions, configurations, or anything that changes over time.

## Decision Signals

**Decomposition is sufficient when** each sub-question can be addressed by targeted lookups that confirm or refute a specific claim — not "learn about a topic."

**Cross-validation is sufficient when** the main conclusion is supported by at least two independent sources, with at least one at the authoritative or established-community tier. If only secondary sources exist, state that explicitly and lower confidence accordingly.

**When sources conflict:** investigate rather than pick a side. Check which is more recent, more authoritative, more specific to your exact context. If the conflict cannot be resolved, report both positions with their evidence.

**Open-ended vs. specific questions:** "What solutions exist for X?" produces a landscape — major options with documented trade-offs. "Does library X support Y?" produces a binary — confirmed or definitively absent from authoritative sources. Judge completeness differently for each.

**Switch from research to prototyping when:** the question is "does this work in practice?" and documentation conflicts; you have two plausible approaches with unclear tradeoffs; or you have been researching an implementation question for 30+ minutes. A 10–30 minute proof-of-concept answers implementation questions faster than more searching. Stay in research for design direction, architecture, tradeoff evaluation, and high-risk domains (security, data integrity, compliance).

For deeper methodology on each of these signals, see `references/epistemic-foundations.md`.

## Failure Signals

Stop and revise when:
- you searched with the original vague question instead of decomposing first
- a search had no declared hypothesis to test — you were exploring, not verifying
- you accepted a secondary source's load-bearing claim without tracing it to a primary source
- you found only confirming evidence and did not actively seek disconfirmation or alternatives
- multiple sources agree but all trace to the same original — you have one piece of evidence, not many
- your conclusion's confidence level does not match the quality and independence of its evidence
- you kept searching after all sub-questions had evidence-backed answers — over-research wastes context
- you are presenting internal knowledge as researched fact without external verification
- you accepted an AI-generated code snippet or API example without testing or verifying against official documentation
- you are researching an implementation question when a prototype would answer it faster

## Completion Criteria

- [ ] The question was decomposed before searching, with a hypothesis or specific purpose per lookup.
- [ ] Load-bearing claims were cross-validated across independent sources.
- [ ] Disconfirming evidence was actively sought.
- [ ] Each conclusion's confidence matches the quality and independence of its supporting evidence.
- [ ] AI-generated content was treated as secondary — verified against official sources when load-bearing.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the question is genuinely answered.

Am I treating a plausible synthesis as if it were well-supported evidence?

Am I stopping because the question is genuinely answered, or because the current evidence looks sufficient enough to move on?

---

See `references/epistemic-foundations.md` for detailed research methodology: confirmation bias, triangulation, provenance tracing, source tiers, cognitive traps, and AI content evaluation.

See `references/research-patterns.md` for completeness standards and common pitfalls in recurring scenarios: library selection, implementation, and best practices.
