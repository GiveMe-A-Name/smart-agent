---
name: systematic-research
description: "Invoke when the task requires gathering information from outside the local codebase and that information cannot be answered from current context."
---

# Systematic Research

Gather external information — web pages, API documentation, external systems — with the same hypothesis-driven discipline applied to code. The goal is grounded conclusions, not comprehensive coverage.

## Trigger Logic

**Invocation default**: Unstructured searching feels productive but rarely produces the most relevant evidence. The cost of invoking unnecessarily is a short framing pass. The cost of skipping is aimless browsing that inflates context and buries the relevant signal. When external information is needed and the answer is not already in context, invoke.

External information means anything outside the local codebase: web search results, official documentation, API references, GitHub issues in external repos, RFCs, external system behavior.

## Capability Boundary

This skill owns:
- decomposing open-ended questions into verifiable sub-questions
- hypothesis-driven lookup execution with declared intent
- annotating conclusions with source quality so downstream judgment can weigh them appropriately

This skill does not own:
- deciding whether gathered information is sufficient for a final decision — that belongs to the skill or workflow using the research
- product clarification or requirement decisions
- investigating the local codebase

## Source Quality

Treat source quality as metadata on every conclusion, not as a filter for what to read.

Three tiers:
- **Authoritative**: official documentation, spec, RFC, changelog, library source code
- **Community**: maintainer-written posts, GitHub issues in the library's own repo, well-established Stack Overflow answers
- **Secondary**: general blog posts, AI-generated summaries, aggregator pages

When a conclusion rests mainly on secondary sources, state that explicitly. Do not present secondary-source findings with the same confidence as authoritative ones. Do not discard secondary sources — they often point toward authoritative sources or reveal the right sub-question to investigate next.

## Invariants

- Decompose the question before any lookup
- Declare the hypothesis each lookup is testing — a lookup without a hypothesis is browsing
- Annotate source quality on every conclusion before exiting
- Stop when all sub-questions are answered, not when curiosity is satisfied
- **Before exiting this skill, you MUST complete the Self-Check section at the end**

## Core Moves

These are judgment moves, not a fixed sequence.

- **Decompose before searching**: convert the original question into specific sub-questions. Do not search with the vague original question.
- **Follow the source hierarchy**: when a secondary source references an authoritative one, follow the reference rather than anchoring the conclusion on the secondary source.
- **Stop when answerable**: when each sub-question has an evidence-backed answer and source quality is annotated, stop.

## Decision Signals

**When is decomposition complete?** When each sub-question can be answered by a single targeted lookup — or when a lookup would confirm or refute a specific claim, not "explore a topic."

**How many lookups?** No fixed limit. A lookup that only reconfirms the same absence in a different location (different page, same missing information) is not a new hypothesis — stop and report the gap instead.

**When is source quality good enough?** When the main conclusion is supported by at least one authoritative or well-established community source. If only secondary sources exist, report that explicitly — do not inflate confidence.

**Open-ended vs. specific questions**: "What solutions exist for X" produces a landscape (known options + documented trade-offs). "Does library X support Y" produces a binary (yes/no with citation). Treat these differently when judging completeness — a landscape is complete when the major options are covered with trade-offs sourced; a binary is complete when confirmed or definitively absent from authoritative sources.

## Failure Signals

Stop and revise when:
- you searched before decomposing the original question
- a lookup had no declared hypothesis
- you followed an interesting tangent that does not test a sub-question from your decomposition
- you treated a secondary source as authoritative without following its references to the original
- you kept searching after all sub-questions had evidence-backed answers
- your conclusion's confidence does not match the source quality behind it

## Self-Check Before Exiting

- [ ] Did I follow all invariants (decompose first, hypothesis per lookup, source quality annotated)?
- [ ] Did I catch any failure signals?
- [ ] Does each conclusion's confidence match its source quality?
- [ ] Am I stopping because the question is genuinely answered, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
