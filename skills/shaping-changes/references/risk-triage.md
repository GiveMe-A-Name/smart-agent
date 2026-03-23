# Risk Triage for `shaping-changes`

Use this only when the main `SKILL.md` is not enough.

## Blocking by Default

Treat these as blocking until adopted, rebutted, or explicitly carried as residual risk:
- `high`: the current change judgment likely breaks the safer path, hides rollback or compatibility danger, misses a shared-contract problem, or leaves a major uncertainty unresolved
- `medium`: the current change judgment is still plausible, but an important boundary, sequencing choice, migration concern, or verification surface is weak enough to change the recommended change shape materially

## Non-Blocking by Default

Treat these as non-blocking unless they combine into something bigger:
- naming preferences
- minor polish
- optional simplifications that do not materially change risk or change quality
- local style disagreements without change-judgment impact

## Escalation Hints

Outside critique is more likely to help when the change touches:
- shared contracts or defaults
- migrations or rollback-sensitive changes
- multiple packages or ownership boundaries
- multiple still-credible approaches with meaningful tradeoffs

Outside critique is less likely to help when the change judgment is:
- local and well-bounded
- single-path and low-risk
- already clear after self-review
