---
name: systematic-research
description: "Gather and validate information from outside the local codebase — docs, APIs, library behavior, external systems. TRIGGER when: the task requires external information not already verified in context. DO NOT TRIGGER when: all needed information is in the local codebase or already verified."
---

# Systematic Research

Use this skill when the task depends on external information that is not already verified in context. External information includes web search results, official documentation, API references, external repositories, RFCs, research papers, expert analysis, package behavior, and external system state.

## Principles

- **Treat internal knowledge as a hypothesis.** Use memory to form questions, not to support conclusions [because training data and recalled facts can be stale, wrong, or mismatched to the user's version].
- **Decompose before searching.** Turn the user's question into specific sub-questions or hypotheses before issuing searches [because vague searches let the first plausible result define the frame and anchor the rest of the research].
- **Research load-bearing claims, not background.** A claim is load-bearing when changing it would change the answer, confidence label, option ranking, exclusion reason, or recommended next check [because equal effort on non-decision claims wastes context while important claims remain under-evidenced].
- **Prefer primary and authoritative sources.** Use official docs, specifications, RFCs, changelogs, library source, peer-reviewed research, project-owned issues/discussions, and official ecosystem comparisons before secondary summaries [because retellings drop caveats, version constraints, and contradictions].
- **Check independence, not citation count.** Treat sources that trace to the same original as one source [because repetition measures propagation, not correctness].
- **Actively look for disconfirmation.** For the leading answer, run at least one lookup or source check that could refute it or compare a live alternative [because supporting-only evidence cannot distinguish research from confirmation bias].
- **Calibrate confidence to evidence.** Report unsupported claims as `Unknown` or `unverified`; do not convert absence of evidence into a fact [because downstream decisions need to know whether a claim is established, weak, conflicted, or unresolved].
- **Switch methods when documentation cannot answer behavior.** If the question is whether something works in practice and sources remain conflicted or inconclusive, prefer a small prototype or executable check in the target environment [because runtime behavior is better evidence than more ambiguous documentation].

## Constraints

- Do not use this skill when all needed information is already available from the local codebase, local tests, local logs, or verified conversation context.
- Before the first search, the notes or response must contain the specific sub-question, hypothesis, or source target being checked.
- Before final response, every load-bearing claim must have a source basis or an explicit `Unknown`/`unverified` marker.
- Before using a secondary source for a load-bearing claim, record the primary/authoritative source it traces to; if none is found, mark the claim `secondary-only`.
- Before counting sources as independent, record that they do not trace to the same original source.
- Do not present AI-generated summaries, snippets, or examples as corroborating evidence; use them only as pointers toward sources or behavior to verify.
- Before assigning `High` confidence, record the authoritative source, independent corroborating source, and any checked conflict status.
- Stop searching when every scoped sub-question has a sourced answer, explicit `Unknown`, or explicit reason for removal from scope.
- Before concluding on a version-sensitive, time-sensitive, or behavior-sensitive claim, record the current authoritative source used for that exact version, date, environment, or behavior.

## Research Decisions

Use these rules to decide what evidence is sufficient.

**Source basis**: For each load-bearing claim, name the source, source tier, date/version relevance when available, and exact claim supported. "Official docs" is not enough; "React API reference, authoritative, current React 19 docs, supports `useActionState` API existence" is enough.

**Source tiers**:
- **Authoritative**: official documentation, specifications, RFCs, changelogs, library source code, peer-reviewed research, official package/framework/foundation comparisons, and project-owned issues or discussions when they include maintainer evidence.
- **Established-community**: maintainer-written posts, accepted/high-score Stack Overflow answers with compatible dates and reproducible evidence, dated technical publications that cite primary sources, and maintained integrations with documented production use.
- **Secondary**: general blogs, tutorials, AI-generated summaries, aggregator pages, forum discussion without primary evidence, and undated summaries.

**Cross-validation**: Treat a main conclusion as established only when at least two independent sources support it and at least one is authoritative or established-community. If only one authoritative source directly answers the question, report `Medium` confidence and mark `not independently corroborated`.

**Confidence labels**:
- `High`: authoritative source plus independent corroboration, no unresolved conflict.
- `Medium`: one authoritative source marked `not independently corroborated`, or two independent established-community sources with no direct contradiction found.
- `Low`: secondary-only, dated, version-unclear, incomplete, or conflicted evidence.
- `Unknown`: no source directly answers the question.

**Conflicts**: When sources conflict, compare recency, authority, specificity to the requested version/environment/scope, and provenance. If the conflict remains unresolved, report the conflict instead of collapsing it into one answer.

**Open-ended questions**: For "what solutions exist for X?", include options found through authoritative sources, established-community discussions, official ecosystem comparisons, or repeated independent adoption signals. Exclude options found only in unsupported secondary mentions unless they affect the user's decision; state the exclusion reason for searched-but-omitted options.

**Specific support questions**: For "does library X support Y?", end with `yes`, `no`, or `unknown`, attach the best source for the exact version, and apply the confidence rules.

**Prototype switch**: Stop researching and run or propose an executable check when documentation conflicts, two implementation approaches remain equally supported after source review, or implementation behavior has consumed 30+ minutes of research without resolving the main uncertainty. Stay in research for design direction, architecture, tradeoff evaluation, and high-risk domains such as security, data integrity, and compliance.

For deeper methodology, see `references/epistemic-foundations.md`. For recurring research scenarios, see `references/research-patterns.md`.

## Examples

**Positive example — external API support question**

User asks whether library X version 4 supports feature Y. Decompose into: current target version; official documentation or changelog for feature Y; source or issue evidence if docs are incomplete; one disconfirming lookup for removed, experimental, or renamed APIs. A complete answer ends `yes`, `no`, or `unknown`, includes the best source for the exact version, and assigns confidence using the label rules above.

**Boundary example — local-codebase question**

User asks where the current project handles client errors. Do not use this skill if the answer can be found by reading local files, call sites, tests, or logs already available in the workspace. External research would add noise because the source of truth is the project code, not web documentation.
