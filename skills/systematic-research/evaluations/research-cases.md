# Systematic Research Evaluation Cases

Use these cases to compare the candidate skill with no skill and with the previous version. They are evaluation inputs, not instructions to load during ordinary research.

## Evaluation Method

Run each condition in a clean context with the same model, tools, and prompt. Repeat runs when practical. Judge the final artifact rather than the presence of named sections or research terminology.

Record:

- conceptual accuracy;
- coverage of important topic-specific perspectives;
- quality of relationships and boundaries in the mental model;
- usefulness of the concrete explanation or example;
- treatment of versions, conflicts, and unknowns;
- prioritization and time-to-understanding;
- calibration of report depth to the user’s wording;
- unsupported claims or misleading compression;
- unnecessary scope expansion;
- tool calls, elapsed time, and output length.

The candidate adds value when it produces a materially clearer or more complete mental model without disproportionate research or reading cost. If baseline and candidate perform equally, remove or challenge the instructions claimed to create the improvement.

## Topic Orientation

### React Server Components

> Research React Server Components. Help an experienced frontend developer quickly understand the concept and its practical implications.

A strong result gives a concise central model, explains the server/client boundary and its relationship to SSR and hydration, and selects only the practical implications that most affect understanding or adoption. It distinguishes stable React concepts from framework behavior without turning the answer into a framework guide, security review, performance handbook, or implementation checklist.

### CRDTs

> Help me quickly understand CRDTs well enough to judge whether they are relevant to a collaborative application.

A strong result builds an intuitive model of convergence and conflict handling, connects it to observable application behavior, and names the few requirements and costs that drive the decision. It should not become an algorithm catalogue, production architecture, test plan, or complete survey of operational concerns.

### WebAssembly Component Model

> Research the WebAssembly Component Model and explain its current significance.

A strong result separates the enduring architectural idea from current proposal, tooling, and adoption status. It explains the problem being solved, the role of interfaces and composition, relationships to core WebAssembly and adjacent standards, and what remains immature or disputed. Current claims must be dated and sourced.

The result should still be a concise orientation. It should not expand every cloud-native use case, runtime, language toolchain, and adoption risk unless the user asks for a comprehensive report.

## Explicit Deep Research

### React Server Components, long form

> Conduct an in-depth investigation of React Server Components for an architecture review. Cover the execution model, relationship to SSR and hydration, framework integration, data and serialization boundaries, caching, security, performance tradeoffs, maturity, and adoption guidance. Produce a comprehensive report with current primary sources.

A strong result expands the requested dimensions, preserves distinctions between React and framework behavior, and provides an executive summary before the detailed analysis. Unlike the default orientation case, additional depth is required here, but unrelated implementation recipes or source-by-source narration are still unnecessary.

## Targeted External Question

### Version-sensitive support

> Does the current stable React release support Activity as a stable API? What qualifications matter?

A strong result answers directly, identifies the exact current version and authoritative source, distinguishes stable, experimental, and framework-specific behavior, and avoids expanding into a general React release history.

## Comparison or Decision

### Data-change architecture

> Compare PostgreSQL logical replication with application-level change events for feeding a search index. Recommend an approach for a small team that needs reliable recovery.

A strong result derives criteria from the stated context, explains the different failure and ownership models, covers recovery and operational consequences, and gives a bounded recommendation. It should not compare products or generic CDC platforms unless they change the decision.

## Near Misses

The skill should remain inactive or sharply bounded for these prompts:

> Where in this repository are React Server Components enabled?

The repository is the source of truth; investigate local code instead.

> Rewrite the following supplied RSC explanation for junior developers.

The material is already provided and the requested capability is rewriting. External research is unnecessary unless the user also asks to verify or update it.

> What does `use client` mean?

Give a concise supported explanation. Do not force a full topic-orientation report when the narrow answer is sufficient.
