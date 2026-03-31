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

## Epistemic Foundations

These are the thinking disciplines that distinguish rigorous research from casual browsing. They are not sequential steps — they are concurrent mental habits that operate throughout the entire research process.

### Think Against Yourself

The greatest threat to research quality is confirmation bias: the tendency to search for, interpret, and recall information in ways that confirm what you already believe (Wason, 1960; Nickerson, 1998). This operates at every stage — which queries you construct, how you read results, which findings you remember.

The countermove is **active disconfirmation**: deliberately seek evidence that would disprove your emerging conclusion. The strength of a conclusion comes not from accumulating supporting evidence, but from surviving genuine attempts to falsify it. When you notice you are only finding confirming evidence, that is a warning signal, not a sign of progress.

As Francis Bacon wrote: "The human understanding when it has once adopted an opinion draws all things else to support and agree with it." Your job is to resist this pull.

### Hold Multiple Hypotheses Simultaneously

Before committing to an answer, generate competing explanations for the same observations (Heuer, Analysis of Competing Hypotheses, CIA, 1999). Evaluate each piece of evidence against all hypotheses, not just the favored one. The goal is to eliminate hypotheses through inconsistency, not to confirm one through selective evidence.

When evidence is consistent with multiple hypotheses, it has low diagnostic value — it tells you less than you think. Seek evidence that differentiates between hypotheses: information that would be expected under one hypothesis but surprising under another.

### Verify Through Convergence, Not Repetition

A claim repeated by ten sources that all trace back to the same original is one piece of evidence, not ten. True validation requires **triangulation**: independent sources, using different methods or data, arriving at the same conclusion (Denzin, 2006).

Types of independence that increase confidence:
- **Source independence**: Different authors/organizations with no shared institutional bias
- **Methodological independence**: Different approaches (e.g., documentation vs. empirical test vs. community experience) reaching the same conclusion
- **Temporal independence**: Consistent findings across different time periods

When sources converge independently, confidence rises. When they diverge, the divergence itself is important data — investigate why rather than picking the answer you prefer.

### Read Laterally, Not Vertically

Professional fact-checkers verify a source before deeply reading its content (Stanford History Education Group). When encountering a new source, leave it to check:
- Who is the author or organization? What is their expertise and track record?
- Do other credible sources corroborate or contradict this claim?
- What is the source's potential motivation or bias?

This is faster and more reliable than deeply analyzing a single source's internal consistency. A beautifully argued page from an unreliable source is still unreliable.

### Trace the Provenance Chain

Claims degrade through retelling. When a secondary source makes a factual claim, trace it to the primary source. Common failure modes:
- Blog posts citing other blog posts, none citing the original documentation
- Outdated information presented as current (check dates)
- Correct information from version X applied to version Y
- AI-generated summaries that confidently state plausible-sounding falsehoods

The primary source may confirm, refute, or add critical nuance that the secondary source lost. Following the chain also reveals when no primary source exists — the claim may be an echo chamber artifact.

### Calibrate Confidence to Evidence Quality

Your confidence in a conclusion must match the quality, independence, and recency of the evidence behind it. Never present a weakly-supported finding with the same certainty as a well-established one.

**Source quality tiers** (treat as metadata on every conclusion):
- **Authoritative**: official documentation, specifications, RFCs, changelogs, library source code, peer-reviewed research. These set the baseline for factual claims.
- **Established Community**: maintainer-written posts, issues/discussions in the library's own repo, well-established Stack Overflow answers with verified solutions, reputable technical publications. These are strong but can contain errors or outdated information.
- **Secondary**: general blog posts, tutorials, AI-generated summaries, aggregator pages, forum discussions. These often point toward authoritative sources or reveal the right question to ask, but should not anchor conclusions on their own.

When a conclusion rests mainly on secondary sources, state that explicitly. When authoritative sources contradict secondary ones, the authoritative source wins absent strong reason to doubt it.

### Recognize Cognitive Traps

Common biases that degrade research quality:
- **Anchoring**: over-weighting the first result you find. The first answer is not the best answer — it is the most available one.
- **Availability bias**: treating easily-found information as more likely to be true or more important.
- **Authority bias**: accepting claims because of who said them rather than the evidence behind them. Authorities can be wrong, especially outside their domain.
- **Illusory truth**: repeated exposure to a claim makes it feel more true, regardless of its actual validity. Popularity is not accuracy.
- **Survivorship bias**: only seeing the successes (working solutions posted online) while missing the failures, edge cases, and caveats that didn't get documented.
- **Premature closure**: stopping research when you find the first plausible answer rather than verifying it and checking alternatives.
- **Conservatism**: insufficiently updating your belief when new evidence contradicts your initial position.

You do not need to formally audit for each bias on every lookup. But when you notice your research feels suspiciously smooth — everything confirming, no contradictions, no surprises — pause and consider whether you are genuinely finding truth or falling into a trap.

### Evaluate AI-Generated Content

AI-generated content (blog posts, StackOverflow answers, documentation, summaries) is now pervasive in search results. It requires specific skepticism beyond general source evaluation.

**Recognition signals:**
- Confident, fluent prose with no hedging, no "I'm not sure," no acknowledged limitations. Real experts hedge; AI-generated text often doesn't.
- Correct-sounding but unverifiable claims. The answer reads well but you can't find the primary source it's implicitly citing.
- Plausible code examples that use incorrect or nonexistent APIs. AI-generated code often looks syntactically valid but references functions, parameters, or behaviors that don't exist in the actual library.
- Generic advice that could apply to any similar question but doesn't address the specific version, configuration, or context you're asking about.

**Handling:**
- Never accept an AI-generated code snippet as correct without testing it or verifying against official documentation. Plausible-looking code is the most dangerous form of misinformation because it compiles in your head.
- When a StackOverflow or blog post answer appears AI-generated, downgrade it to the lowest source tier. It may contain useful keywords or point you toward the right documentation, but treat it as a search hint, not an answer.
- AI-generated summaries of documentation often lose critical nuance — edge cases, version-specific behavior, caveats. Always check the original documentation when the claim matters.

### When to Stop Researching and Build Instead

Research has diminishing returns. Some questions are better answered by building a small experiment than by reading more.

**Switch from research to prototyping when:**
- The question is "does this work in practice?" and the documentation says it should but you've found conflicting reports. A 30-minute proof-of-concept answers this faster than another hour of reading.
- You have two plausible approaches and the tradeoffs are unclear from documentation alone. Build the simplest version of each and measure.
- The documentation is sparse, outdated, or contradictory, and the library is available locally. Reading the source code or running experiments against it is more reliable than searching for someone else's interpretation.
- You've found 3+ sources that agree on the general approach but disagree on implementation details (exact API calls, configuration keys, parameter values). The details are best verified by running them.

**Stay in research when:**
- The question is about design direction, architecture, or tradeoff evaluation. You can't prototype "should we use a message queue vs direct API calls" in 30 minutes — you need to understand the tradeoffs first.
- The risk of getting it wrong is high (security, data integrity, compliance). Prototyping in these domains without sufficient background research is dangerous.
- You haven't yet formed a hypothesis. Prototyping without a thesis is just random experimentation.

## Invariants

- Decompose the question before searching — a vague question produces vague results
- Every search should test a hypothesis or answer a specific sub-question, not "explore a topic"
- Cross-validate important claims across independent sources before treating them as established
- Trace secondary-source claims to their primary source when the claim is load-bearing for your conclusion
- Annotate source quality and confidence level on every conclusion
- Distinguish between "I found evidence that X is true" and "I found no evidence that X is false" — absence of evidence is not evidence of absence
- Use web search tools actively — do not rely solely on internal knowledge for factual claims about external systems, libraries, APIs, or current state of the world
- Stop when sub-questions are answered with appropriate evidence, not when curiosity is satisfied or a time limit is hit
- **Before exiting, complete the Self-Check section**

## Decision Signals

**When is decomposition good enough?** When each sub-question can be addressed by targeted lookup(s) that would confirm or refute a specific claim — not "learn about a topic."

**How many searches are enough?** There is no fixed limit. But a search that only reconfirms the same absence from a different location is not a new hypothesis — report the gap instead. Conversely, if you have only one source for a critical claim, you are not done.

**When is cross-validation sufficient?** When the main conclusion is supported by at least two independent sources, with at least one at the authoritative or established community tier. If only secondary sources exist, report that explicitly and lower your confidence accordingly.

**When sources conflict:** Conflicts between sources are high-value signals. Investigate the conflict rather than picking a side:
- Check which source is more recent, more authoritative, more specific to the exact context
- Check whether the conflict is real (different versions, different configurations, different edge cases) or apparent (different terminology for the same thing)
- If the conflict cannot be resolved, report both positions with their evidence and let the consumer decide

**Open-ended vs. specific questions:** "What solutions exist for X?" produces a landscape — major options with documented trade-offs, sourced. "Does library X support Y?" produces a binary — yes/no with citation. Judge completeness differently: a landscape is complete when major options are covered with sourced trade-offs; a binary is complete when confirmed or definitively absent from authoritative sources.

**When to distrust your own knowledge:** Always, for factual claims about versions, APIs, configuration, compatibility, or anything that changes over time. Internal knowledge is useful for framing questions and generating hypotheses, but external verification is required before presenting such claims as facts.

## Common Research Patterns

These are recurring research scenarios in software engineering. Each has a distinct completeness standard and common pitfalls.

### "Which library/tool should we use?"

**Decompose into:** What problem are we solving? What are the top 3-5 options? For each: maintenance health, community size, security history, API quality, performance characteristics, license. What are the known failure modes or limitations?

**Completeness standard:** Major options covered with sourced tradeoffs. At least one authoritative source per option (official docs, maintainer posts). At least one critical/negative review per top candidate (guard against survivorship bias).

**Common pitfall:** Evaluating libraries by their README quality or GitHub stars. Stars measure popularity, not quality. README quality measures marketing, not reliability.

### "How do I implement X with library Y?"

**Decompose into:** What does the official documentation say? What version am I using? Are there known issues or breaking changes in recent versions? What do working examples in the library's own test suite or examples folder show?

**Completeness standard:** Verified against official docs for the specific version. If the official docs are insufficient, check the library's GitHub issues and discussions for the specific use case.

**Common pitfall:** Following a blog tutorial written for version 2.x when you're on version 4.x. Always check the publication date and target version of any tutorial or guide.

### "What's the best practice for X?"

**Decompose into:** Best practice according to whom? In what context? What tradeoffs does this "best practice" make, and are those tradeoffs appropriate for our situation?

**Completeness standard:** At least two independent sources that agree, with at least one explaining the reasoning behind the practice (not just stating it). If the "best practice" involves a tradeoff, both sides of the tradeoff should be represented.

**Common pitfall:** Treating a practice as universal when it's context-dependent. "Always use an ORM" is not a best practice — it's a recommendation for certain contexts with specific tradeoffs.

## Failure Signals

Stop and revise when:
- You searched with the original vague question instead of decomposing it first
- A search had no declared purpose — you were browsing, not testing
- You followed an interesting tangent that does not answer any of your sub-questions
- You accepted a secondary source's claim without checking whether it traces to a primary source, and the claim is load-bearing
- You found only confirming evidence and did not search for disconfirming evidence
- Multiple sources agree but all trace to the same original — you have one source, not many
- You treated a popular answer as correct without checking its actual validity
- Your conclusion's confidence level does not match the evidence quality behind it
- You stopped at the first plausible answer without checking for alternatives or edge cases
- You kept searching after all sub-questions had evidence-backed answers (over-research wastes context)
- You are presenting your internal knowledge as researched fact without external verification
- You accepted an AI-generated code snippet or API example without testing or verifying against official documentation
- You have been researching an implementation question for 30+ minutes when a 10-minute prototype would answer it definitively
- You are evaluating a library by its GitHub stars, README aesthetics, or a single glowing blog post instead of maintenance health, issue response time, and security history

## Self-Check Before Exiting

- [ ] Did I decompose before searching, with a hypothesis or purpose per lookup?
- [ ] Did I cross-validate critical claims across independent sources?
- [ ] Did I actively search for disconfirming evidence, not just confirming evidence?
- [ ] Does each conclusion's confidence match the quality and independence of its supporting evidence?
- [ ] Did I evaluate AI-generated content with appropriate skepticism (verified code examples, checked original docs)?
- [ ] If an implementation question: did I consider whether a prototype would be faster than more research?
- [ ] Am I stopping because the question is genuinely answered, or rationalizing early closure?

**If any check fails, return to the relevant section before exiting.**
