---
name: shaping-changes
description: Use when the requirement and repository slice are already grounded, but the agent still needs judgment about which change is better, where it should begin and end, which risks materially matter, or whether the understanding is strong enough to proceed into implementation.
---

# Shaping Changes

Shape a better change before implementation starts.

This skill teaches change judgment: compare credible paths, choose a safer
change shape, set boundaries, surface change-shaping risks, and decide whether
the work is clear enough to move into implementation. It does not teach
implementation design, task breakdown, or code-writing order.

## Trigger Logic

Use this skill when:
- the requirement and repository slice are already grounded
- multiple change paths are credible, or the change boundary still needs judgment
- rollback, migration, compatibility, ownership, shared-contract, or default-behavior risk could change the safer path
- the main question is "what is the better change to make?" rather than "how should it be implemented?"

Do not use this skill when:
- the task is a small, local, obvious edit with one clear low-risk path
- the main missing step is requirement clarification, repository understanding, implementation design, implementation, or completion judgment
- the work is purely documentation, comment, or copy editing

## Boundary

This skill owns:
- choosing the better change approach from grounded alternatives
- deciding where the change should begin and end
- identifying risks and unknowns that materially affect the decision
- deciding whether the current understanding is strong enough to proceed
- deciding whether self-review is sufficient or outside critique is warranted

This skill does not own:
- clarifying product intent from scratch
- doing repository-slice analysis as the main output
- designing the implementation in detail
- breaking implementation into coding tasks or file edits
- implementing the change
- deciding whether the implementation is complete

## Core Change Questions

A strong change judgment can answer these questions clearly:
- What problem is being solved, and what constraint makes this non-trivial?
- What are the credible paths, and why is one stronger here?
- Where should the change begin and end?
- If order matters for safety, migration, rollback, or compatibility, what sequence reduces risk and confusion?
- Which risks could materially change the decision?
- What must later implementation preserve or demonstrate?
- What remains uncertain?

If the change judgment cannot answer these, it is probably still analysis,
implementation drift, or a premature execution sketch rather than real change judgment.

## Judgment Heuristics

- Compare alternatives only when they are genuinely credible; do not invent fake options.
- Prefer the smallest effective change that still respects ownership and future maintenance.
- Compare change shapes, not the first editable files or the easiest first step.
- Discuss sequence only when order materially affects rollback, migration, compatibility, or shared contracts.
- Treat shared contracts, defaults, migrations, and cross-package coupling as change-shaping risks.
- Keep later verification in view, but do not collapse change judgment into a test checklist or implementation design.
- Distinguish proven facts, change assumptions, and open unknowns.

## Review and Escalation Principles

Default to self-review first.

Outside critique is warranted when unresolved risk or uncertainty is high enough
that another perspective is likely to change the recommended change shape materially, especially for
shared contracts, rollback-sensitive changes, migrations, architecture boundary
changes, or multiple still-credible approaches.

Do not escalate merely because a change is "non-trivial" in a generic sense.

When outside review happens:
- focus on medium/high-risk problems first
- require explicit disposition for medium/high-risk findings before treating the change judgment as ready
- treat low-risk findings as non-blocking by default unless they aggregate into a more serious issue
- do not let preference debates, polish, or minor optional improvements block readiness

## Self-Correction Signals

Stop and revise when:
- you are shaping changes on top of unclear intent or unclear ownership
- you are describing modules, files, or edit order instead of explaining the better change
- you are choosing the first plausible path without real comparison
- you are turning the judgment into an implementation sketch, task breakdown, or final completion judgment
- you are escalating to subagent review before self-review
- you are letting low-risk review findings block readiness
- you are calling the change judgment ready while unresolved medium/high-risk findings still stand

## What Good Looks Like

A good result:
- turns grounded understanding into a concrete change judgment
- compares real alternatives when they matter
- explains boundaries, risks, and relevant order constraints without becoming implementation design
- keeps change judgment separate from implementation design and completion judgment
- uses self-review by default and outside review selectively
- treats unresolved medium/high-risk review findings as readiness blockers
- keeps low-risk findings non-blocking unless they reveal a broader flaw

If the result mainly reads like a file list, an execution script, or an issue
dump, the change-judgment capability is weak.

## Supporting References

- Read `references/risk-triage.md` for expanded review severity guidance.
