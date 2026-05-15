---
name: systematic-research
description: "Gather and validate information from outside the local codebase — docs, APIs, library behavior, external systems. TRIGGER when: the task requires external information not already verified in context. DO NOT TRIGGER when: all needed information is in the local codebase or already verified."
---

# Systematic Research

Gather, evaluate, and synthesize external information so each conclusion can be traced to observable evidence: a named source basis, an independence check for load-bearing claims, and an explicit confidence or `unknown` marker. The goal is grounded, cross-validated conclusions — not comprehensive coverage, not speed, and not confirming what you already believe.

Example of sufficient evidence framing: "React 19 supports `useActionState` — confidence: `High` — source basis: official React API reference plus the React 19 release post; independent check: one established-community migration guide confirms the same API surface." Example of insufficient framing: "React 19 probably supports the new hook because several blog posts mention it" — no authoritative source, no independence check, and no confidence rule applied.

Your internal knowledge is a starting point for questions, never a substitute for evidence. Treat it as an untested hypothesis until corroborated by external sources.

## Trigger Logic

**Invocation default**: Unstructured searching produces anchored, confirmation-biased results when no lookup has a declared hypothesis or source target. The cost of invoking unnecessarily is a short framing pass. The cost of skipping is conclusions that sound authoritative but rest on unverified assumptions, single sources, or stale knowledge. When external information is needed and the answer is not already supported by verified evidence in context, invoke.

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
- making the final product, architecture, or implementation decision from the research; the synthesis may state evidence-backed tradeoffs or recommended next checks, but must not present the decision as settled by research alone
- defining new product requirements or priorities
- investigating behavior that can be answered from the local codebase alone

## Invariants

- Never issue a search with the original vague question — decompose into specific sub-questions first, each targeting a verifiable hypothesis. [Because vague queries produce vague results, and the first result anchors all subsequent searches into a confirmation bias spiral. Specific queries test specific hypotheses and produce falsifiable answers.]
- Cross-validate every load-bearing claim across independent sources before treating it as established or assigning `High` confidence. Sources that all trace back to the same original count as one, not many. A claim supported by one authoritative source may be reported with `Medium` confidence only if it is marked `not independently corroborated`. [Because echo chambers are indistinguishable from consensus without checking independence. Ten sources citing the same blog post are one piece of evidence — not ten.]
- Trace every secondary-source claim to its primary source when the conclusion depends on it. [Because claims degrade through retelling — secondary sources drop caveats, version specifics, and contrary evidence the primary source contained. Tracing the chain also reveals when no primary source exists at all.]
- If you cannot verify a claim externally, mark it as `unverified` or `unknown`; do not restate it as fact. [Because confidence must be calibrated to evidence quality. Presenting unverified claims as established facts misleads every downstream decision that depends on this research.]
- For any version-sensitive, time-sensitive, or behavior-sensitive claim, inspect at least one external authoritative source before concluding. [Because correct information from version X applied to version Y is a research failure. Confidently stated stale information is more dangerous than acknowledged uncertainty.]

## Decision Signals

**Decomposition is sufficient when** each sub-question can be addressed by targeted lookups that confirm or refute a specific claim — not "learn about a topic."

**Load-bearing claims:** a claim is load-bearing when changing it would change the final answer, confidence label, option ranking, exclusion reason, or recommended next check. Claims that only provide background or wording support are not load-bearing unless the final answer depends on them.

**Source basis:** a source basis names the source, its tier, date/version relevance when available, and the exact claim it supports. "Official docs" alone is not a source basis; "React API reference, authoritative, current React 19 docs, supports `useActionState` API existence" is.

**Source tiers:** authoritative means official documentation, specifications, RFCs, changelogs, library source code, peer-reviewed research, or official ecosystem comparisons published by the project, framework, foundation, or package-registry ecosystem. Established-community means maintainer-written posts; issues/discussions in the project-owned repository; Stack Overflow answers only when the answer is accepted or has score ≥ 10, the answer date is compatible with the target version, comments do not report current breakage, and the answer links to official docs/source or includes reproducible evidence; technical publications only when they name an author or organization, include a publication/update date, cite primary docs/source or include reproducible data, and are not AI-generated summaries or aggregator pages; or maintained integrations with documented production use. Secondary means general blogs, tutorials, AI-generated summaries, aggregator pages, and forum discussions. A repeated independent adoption signal requires at least two independent production-use reports, case studies, or maintained integrations that do not cite the same original source.

**Cross-validation is sufficient when** the main conclusion is supported by at least two independent sources, with at least one at the authoritative or established-community tier. If only one authoritative source directly answers the question, the claim may be reported as `Medium` and `not independently corroborated`; it is not established or `High`. If only secondary sources exist, state that explicitly and lower confidence accordingly.

**Confidence labels are assigned from evidence, not tone:** use `High` only when an authoritative source plus independent corroboration supports the conclusion and no unresolved conflict was found; `Medium` when one authoritative source supports it and the conclusion is marked `not independently corroborated`, or when two independent established-community sources support it with no direct contradiction found; `Low` when evidence is secondary-only, dated, version-unclear, or conflicted; `Unknown` when no source directly answers the question.

**When sources conflict:** compare recency, authority, and specificity to the exact version, environment, or scope in question. If the conflict remains unresolved, report the conflict explicitly and do not collapse it into a single answer.

**Open-ended vs. specific questions:** for "What solutions exist for X?", completeness means naming every option found through the declared inclusion signals: authoritative sources, established-community discussions, official ecosystem comparisons, or repeated independent adoption signals. Exclude options found only in secondary or unsupported mentions unless they affect the user's decision; when a searched option is omitted, state the exclusion reason. Each included option needs at least one sourced tradeoff or limitation. For "Does library X support Y?", completeness means the answer is stated as `yes`, `no`, or `unknown`, with the best available source attached.

**Switch from research to prototyping when:** the question is "does this work in practice?" and documentation conflicts; two implementation approaches still each have at least one supporting source and no decisive contradiction after source review; or an implementation question has consumed 30+ minutes of research without resolving the main uncertainty. Stay in research for design direction, architecture, tradeoff evaluation, and high-risk domains (security, data integrity, compliance).

For deeper methodology on each of these signals, see `references/epistemic-foundations.md`.

## Failure Signals

Stop and revise when:
- you searched with the original vague question instead of decomposing first [because the first plausible result can define the frame before any hypothesis has been made falsifiable]
- a search had no declared hypothesis to test — exploratory search lets the first plausible result define the frame, so later evidence gathering becomes confirmation rather than verification
- you accepted a secondary source's load-bearing claim without tracing it to a primary source [because retellings can drop version constraints, caveats, and contradictory details]
- no lookup or source check attempted to falsify the leading hypothesis or compare a live alternative [because supporting-only evidence cannot distinguish truth from confirmation bias]
- multiple sources agree but all trace to the same original — you have one piece of evidence, not many [because repetition without independence measures propagation, not correctness]
- a load-bearing conclusion has no explicit confidence level, source basis, or `unknown` marker [because downstream decisions cannot tell whether the claim is established, weak, or unresolved]
- you kept searching after all sub-questions had evidence-backed answers [because over-research wastes context and can obscure the answer with low-value extra material]
- you are presenting internal knowledge as researched fact without external verification [because memory and training data can be stale, wrong, or mismatched to the user's version]
- you accepted an AI-generated code snippet or API example without testing or verifying against official documentation [because plausible code can reference nonexistent APIs while still looking syntactically correct]
- you are researching an implementation question when a prototype would answer it faster [because documentation ambiguity about runtime behavior is lower-quality evidence than an executable check in the target environment]

## Completion Criteria

- [ ] The question was decomposed before searching, with a hypothesis or specific purpose per lookup.
- [ ] Load-bearing claims assigned `High` or treated as established were cross-validated across independent sources; single-authoritative-source claims were labeled `Medium` and marked `not independently corroborated`.
- [ ] At least one lookup or source check attempted to falsify the leading hypothesis or compare an alternative found in authoritative sources, established-community sources, official ecosystem comparisons, or repeated independent adoption signals.
- [ ] Each load-bearing conclusion is labeled `High`, `Medium`, `Low`, or `Unknown` according to the confidence-label rules, and is supported by a source basis or explicitly marked `unknown`/`unverified`.
- [ ] Open-ended questions name the options meeting the inclusion signals and their sourced tradeoffs, with exclusion reasons for searched-but-omitted options; specific questions end in `yes`, `no`, or `unknown`.
- [ ] AI-generated content was treated as secondary — verified against official sources when load-bearing.

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is artifact-type: the artifact is a research synthesis whose claims can be checked against recorded sub-questions, source bases, provenance, and confidence labels. Completion means every Completion Criteria item can be answered by pointing to specific sources with provenance — not from synthesis or memory, but from citable evidence. The verification is not "I believe the research is thorough" but "I can point to the primary or established-community source for each load-bearing claim."

Research that looks complete is not the same as research that is complete. A findings document can have correct structure, confidence labels, and source citations while resting on a single secondary source or internal knowledge presented as external evidence. The checkable evidence is the source list itself — not the confidence level assigned.

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to compare the final synthesis against the recorded sub-questions, lookups, source bases, and confidence labels.

This check exists because research with many citations, confidence labels, and polished synthesis can still rest on unverified assumptions. The fluency of the output is not evidence of the quality of the evidence behind it.

Does any conclusion appear in the synthesis without a corresponding source basis in the source list or conversation evidence?

Am I stopping while any original sub-question lacks one of: a cited answer, an explicit `unknown` marker, or a reason it was removed from scope?

Completion-faking signals specific to systematic research — stop if any apply:

- A conclusion sounds authoritative and sourced, but all supporting sources were secondary tier — no authoritative source (official documentation, spec, RFC, library source) was actually consulted
- Internal knowledge was used to fill a gap in external evidence, then presented as if that gap was addressed by research
- Multiple sources agreed, and that convergence was counted as cross-validation — but all those sources cited the same original post, documentation page, or blog entry
- Research stopped while any claim assigned `High` or treated as established lacked cross-validation against independent sources, or while a single-authoritative-source claim lacked the `Medium` + `not independently corroborated` marker
- AI-generated content on the topic was treated as corroborating evidence rather than as a secondary signal pointing toward authoritative sources that need to be read directly

---

See `references/epistemic-foundations.md` for detailed research methodology: confirmation bias, triangulation, provenance tracing, source tiers, cognitive traps, and AI content evaluation.

See `references/research-patterns.md` for completeness standards and common pitfalls in recurring scenarios: library selection, implementation, and best practices.

## Examples

**Positive example — external API support question:** User asks whether library X version 4 supports feature Y. Decompose into: current project/library version; official documentation or changelog for feature Y; source or issue evidence if docs are incomplete; one disconfirming lookup for removed, experimental, or renamed APIs. A complete answer ends `yes`, `no`, or `unknown`, includes the best source for the exact version, and assigns confidence using the label rules above.

**Boundary example — local-codebase question:** User asks where the current project handles client errors. Do not use this skill if the answer can be found by reading local files, call sites, tests, or logs already available in the workspace. External research would add noise because the source of truth is the project code, not documentation or web search.
