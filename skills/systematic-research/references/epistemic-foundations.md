# Epistemic Foundations

Evidence-handling constraints that distinguish systematic research from casual browsing. These are not sequential steps; they are observable checks to apply throughout research: what hypothesis each lookup tests, whether a source is independent, whether a claim traces to a primary source, and whether confidence matches evidence quality.

---

## Think Against Yourself

The greatest threat to research quality is confirmation bias: the tendency to search for, interpret, and recall information in ways that confirm what you already believe (Wason, 1960; Nickerson, 1998). This operates at every stage — which queries you construct, how you read results, which findings you remember.

The countermove is **active disconfirmation**: deliberately seek evidence that would disprove your emerging conclusion. The strength of a conclusion comes not from accumulating supporting evidence, but from surviving genuine attempts to falsify it. When you notice you are only finding confirming evidence, that is a warning signal, not a sign of progress.

As Francis Bacon wrote: "The human understanding when it has once adopted an opinion draws all things else to support and agree with it." Translate that risk into an action: before final synthesis, include at least one lookup or source check that would have weakened the leading conclusion if the contrary evidence existed.

---

## Hold Multiple Hypotheses Simultaneously

Before committing to an answer, generate competing explanations for the same observations (Heuer, Analysis of Competing Hypotheses, CIA, 1999). Evaluate each piece of evidence against all hypotheses, not just the favored one. The goal is to eliminate hypotheses through inconsistency, not to confirm one through selective evidence.

When evidence is consistent with multiple hypotheses, it has low diagnostic value — it tells you less than you think. Seek evidence that differentiates between hypotheses: information that would be expected under one hypothesis but surprising under another.

---

## Verify Through Convergence, Not Repetition

A claim repeated by ten sources that all trace back to the same original is one piece of evidence, not ten. True validation requires **triangulation**: independent sources, using different methods or data, arriving at the same conclusion (Denzin, 2006).

Types of independence that increase confidence:
- **Source independence**: Different authors/organizations with no shared institutional bias
- **Methodological independence**: Different approaches (e.g., documentation vs. empirical test vs. community experience) reaching the same conclusion
- **Temporal independence**: Consistent findings across different time periods

When sources converge independently, confidence rises. When they diverge, the divergence itself is important data — investigate why rather than picking the answer you prefer.

---

## Read Laterally, Not Vertically

Professional fact-checkers verify a source before deeply reading its content (Stanford History Education Group). When encountering a new source, leave it to check:
- Who is the author or organization? What is their expertise and track record?
- Do other credible sources corroborate or contradict this claim?
- What is the source's potential motivation or bias?

This is faster and more reliable than deeply analyzing a single source's internal consistency. A beautifully argued page from an unreliable source is still unreliable.

---

## Trace the Provenance Chain

Claims degrade through retelling. When a secondary source makes a factual claim, trace it to the primary source. Common failure modes:
- Blog posts citing other blog posts, none citing the original documentation
- Outdated information presented as current (check dates)
- Correct information from version X applied to version Y
- AI-generated summaries that confidently state plausible-sounding falsehoods

The primary source may confirm, refute, or add critical nuance that the secondary source lost. Following the chain also reveals when no primary source exists — the claim may be an echo chamber artifact.

---

## Calibrate Confidence to Evidence Quality

Your confidence in a conclusion must match the quality, independence, and recency of the evidence behind it. Never present a weakly-supported finding with the same certainty as a claim supported by authoritative sources plus independent corroboration.

**Source quality tiers** (treat as metadata on every conclusion):
- **Authoritative**: official documentation, specifications, RFCs, changelogs, library source code, peer-reviewed research. These set the baseline for factual claims.
- **Established Community**: maintainer-written posts; issues/discussions in the library's own repo; Stack Overflow answers only when the answer is accepted or has score ≥ 10, the answer date is compatible with the target version, comments do not report current breakage, and the answer links to official docs/source or includes reproducible evidence; technical publications only when they name an author or organization, include a publication/update date, cite primary docs/source or include reproducible data, and are not AI-generated summaries or aggregator pages. These are strong but can contain errors or outdated information.
- **Secondary**: general blog posts, tutorials, AI-generated summaries, aggregator pages, forum discussions. These often point toward authoritative sources or reveal the right question to ask, but should not anchor conclusions on their own.

When a conclusion rests mainly on secondary sources, state that explicitly. When authoritative sources contradict secondary ones, the authoritative source wins absent strong reason to doubt it.

---

## Recognize Cognitive Traps

Common biases that degrade research quality:
- **Anchoring**: over-weighting the first result you find. The first answer is not the best answer — it is the most available one.
- **Availability bias**: treating easily-found information as more likely to be true or more important.
- **Authority bias**: accepting claims because of who said them rather than the evidence behind them. Authorities can be wrong, especially outside their domain.
- **Illusory truth**: repeated exposure to a claim makes it feel more true, regardless of its actual validity. Popularity is not accuracy.
- **Survivorship bias**: only seeing the successes (working solutions posted online) while missing the failures, edge cases, and caveats that didn't get documented.
- **Premature closure**: stopping research when you find the first plausible answer rather than verifying it and checking alternatives.
- **Conservatism**: insufficiently updating your belief when new evidence contradicts your initial position.
- **Completeness theater**: the appearance of thorough research — multiple lookups performed, sources cited, confidence labels attached — without the actual cross-validation those signals imply. A findings document that looks well-sourced is more persuasive than one acknowledged as partial, which is precisely why this failure mode is dangerous. The check: can you point to the primary source for each load-bearing claim? If not, the confidence label is borrowed from the structure of the output, not earned by the evidence behind it.

When every source found so far supports the same conclusion and no query has targeted disconfirming evidence, stop before synthesis and run a falsifying or alternative-comparison lookup. Agreement without attempted disconfirmation is not evidence that no contradiction exists.

---

## Evaluate AI-Generated Content

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
