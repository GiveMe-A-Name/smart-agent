---
name: systematic-research
description: "Gather and validate information from outside the local codebase — docs, APIs, library behavior, external systems. TRIGGER when: the task requires external information not already verified in context. DO NOT TRIGGER when: all needed information is in the local codebase or already verified."
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
- making the final product, architecture, or implementation decision from the research
- defining new product requirements or priorities
- investigating behavior that can be answered from the local codebase alone

## Invariants

- Never issue a search with the original vague question — decompose into specific sub-questions first, each targeting a verifiable hypothesis. [Because vague queries produce vague results, and the first result anchors all subsequent searches into a confirmation bias spiral. Specific queries test specific hypotheses and produce falsifiable answers.]
- Cross-validate every load-bearing claim across independent sources before treating it as established. Sources that all trace back to the same original count as one, not many. [Because echo chambers are indistinguishable from consensus without checking independence. Ten sources citing the same blog post are one piece of evidence — not ten.]
- Trace every secondary-source claim to its primary source when the conclusion depends on it. [Because claims degrade through retelling — secondary sources drop caveats, version specifics, and contrary evidence the primary source contained. Tracing the chain also reveals when no primary source exists at all.]
- If you cannot verify a claim externally, mark it as `unverified` or `unknown`; do not restate it as fact. [Because confidence must be calibrated to evidence quality. Presenting unverified claims as established facts misleads every downstream decision that depends on this research.]
- For any version-sensitive, time-sensitive, or behavior-sensitive claim, inspect at least one external authoritative source before concluding. [Because correct information from version X applied to version Y is a research failure. Confidently stated stale information is more dangerous than acknowledged uncertainty.]

## Decision Signals

**Decomposition is sufficient when** each sub-question can be addressed by targeted lookups that confirm or refute a specific claim — not "learn about a topic."

**Cross-validation is sufficient when** the main conclusion is supported by at least two independent sources, with at least one at the authoritative or established-community tier. If only secondary sources exist, state that explicitly and lower confidence accordingly.

**When sources conflict:** compare recency, authority, and specificity to the exact version, environment, or scope in question. If the conflict remains unresolved, report the conflict explicitly and do not collapse it into a single answer.

**Open-ended vs. specific questions:** for "What solutions exist for X?", completeness means the serious options are named and each has at least one sourced tradeoff or limitation. For "Does library X support Y?", completeness means the answer is stated as `yes`, `no`, or `unknown`, with the best available source attached.

**Switch from research to prototyping when:** the question is "does this work in practice?" and documentation conflicts; two implementation approaches remain plausible after source review; or an implementation question has consumed 30+ minutes of research without resolving the main uncertainty. Stay in research for design direction, architecture, tradeoff evaluation, and high-risk domains (security, data integrity, compliance).

For deeper methodology on each of these signals, see `references/epistemic-foundations.md`.

## Failure Signals

Stop and revise when:
- you searched with the original vague question instead of decomposing first
- a search had no declared hypothesis to test — you were exploring, not verifying
- you accepted a secondary source's load-bearing claim without tracing it to a primary source
- no lookup or source check attempted to falsify the leading hypothesis or compare a live alternative
- multiple sources agree but all trace to the same original — you have one piece of evidence, not many
- a load-bearing conclusion has no explicit confidence level, source basis, or `unknown` marker
- you kept searching after all sub-questions had evidence-backed answers — over-research wastes context
- you are presenting internal knowledge as researched fact without external verification
- you accepted an AI-generated code snippet or API example without testing or verifying against official documentation
- you are researching an implementation question when a prototype would answer it faster

## Completion Criteria

- [ ] The question was decomposed before searching, with a hypothesis or specific purpose per lookup.
- [ ] Load-bearing claims were cross-validated across independent sources.
- [ ] At least one lookup or source check attempted to falsify the leading hypothesis or compare a serious alternative.
- [ ] Each load-bearing conclusion is labeled with confidence and supported by a source basis, or is explicitly marked `unknown`/`unverified`.
- [ ] Open-ended questions name the serious options and their sourced tradeoffs; specific questions end in `yes`, `no`, or `unknown`.
- [ ] AI-generated content was treated as secondary — verified against official sources when load-bearing.

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is evidence-accumulation-type: completion means every Completion Criteria item can be answered by pointing to specific sources with provenance — not from synthesis or memory, but from citable evidence. The verification is not "I believe the research is thorough" but "I can point to the primary or established-community source for each load-bearing claim."

Research that looks complete is not the same as research that is complete. A findings document can have correct structure, confidence labels, and source citations while resting on a single secondary source or internal knowledge presented as external evidence. The checkable evidence is the source list itself — not the confidence level assigned.

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the question is genuinely answered.

This check exists because research that feels thorough is not the same as research that is thorough. A synthesis can sound authoritative and well-organized while resting on unverified assumptions — the fluency of the output is not evidence of the quality of the evidence behind it.

Am I treating a plausible synthesis as if it were well-supported evidence?

Am I stopping because the question is genuinely answered, or because the current evidence looks sufficient enough to move on?

Completion-faking signals specific to systematic research — stop if any apply:

- A conclusion sounds authoritative and sourced, but all supporting sources were secondary tier — no authoritative source (official documentation, spec, RFC, library source) was actually consulted
- Internal knowledge was used to fill a gap in external evidence, then presented as if that gap was addressed by research
- Multiple sources agreed, and that convergence was counted as cross-validation — but all those sources cited the same original post, documentation page, or blog entry
- Research stopped when the answer felt plausible rather than when all load-bearing claims were cross-validated against independent sources
- AI-generated content on the topic was treated as corroborating evidence rather than as a secondary signal pointing toward authoritative sources that need to be read directly

---

See `references/epistemic-foundations.md` for detailed research methodology: confirmation bias, triangulation, provenance tracing, source tiers, cognitive traps, and AI content evaluation.

See `references/research-patterns.md` for completeness standards and common pitfalls in recurring scenarios: library selection, implementation, and best practices.
