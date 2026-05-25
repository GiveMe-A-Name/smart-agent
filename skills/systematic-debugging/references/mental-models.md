# Mental Models — Full Detail

Extended explanations for the five mental models summarized in SKILL.md. Use this file when a model must be applied deeply, when debugging has stopped eliminating possibilities, or when a stuck-loop signal appears.

---

## Debugging Is a Search Problem

A bug is one broken thing among thousands of working things. Every debugging action should eliminate at least one named possibility. If an action does not shrink the search space, change the action.

Before acting, choose the fastest experiment that can eliminate the most remaining plausible causes.

When the search space is large, use **divide and conquer**: split it in half, test which half contains the bug, discard the other, repeat. This gives logarithmic convergence — even in a million-line codebase, ~20 well-chosen binary splits find the bug.

Ways to split:
- **Through code**: checkpoint halfway through execution. Data correct at midpoint? Bug is in the second half.
- **Through time**: `git bisect`. "Worked Tuesday, broken now" → bisect finds the exact breaking commit.
- **Through components**: in layered systems (UI → API → Service → DB), check data at each boundary. Bug lives between the last correct boundary and the first incorrect one.
- **Through input**: some inputs trigger it, others don't. Bisect the input space — which field, which value range?

---

## Observation Over Theory

Form hypotheses only after recording observations. A theory formed before data can anchor the diagnosis to a false explanation.

What observation looks like in practice:
- Read the **full** error message: type, full stack trace, first project-owned frame, and data values. "What the error says" and "where the project code originated it" are different questions.
- **Print the actual value.** Do not rely on assumed variable, fixture, config, or environment state.
- **Instrument at boundaries.** In multi-component systems, add logging at each boundary, run once, read where data breaks. One instrumented run costs less than three wrong hypotheses.
- **Find a working example** of the same pattern and diff it against the broken case. List every difference, however small.

---

## Symptom Site ≠ Owning Layer

The file or line where the error surfaces is where the symptom appears, not necessarily where the fix belongs. A stack trace points to the symptom site; the owning layer requires tracing the behavior back to its source.

When an error appears at `file.ts:42`, first ask what put the system in a state where line 42 fails. The owning layer can be a different file, layer, or repository.

Before accepting the stack-trace location as the bug, check the competing explanations that are plausible in this case:
- **Expectation wrong?** Is the assertion itself testing for the wrong thing?
- **Fixture wrong?** Is the test setup producing bad input or state?
- **Environment wrong?** If config, timing, deps, build state, or tooling could reasonably explain it, is something outside the code causing this?
- **Implementation wrong?** Is the production code actually at fault?

Check each plausible explanation before committing to one.

---

## Causal Chain Depth

Every surface cause has a deeper cause. Fix depth determines whether the bug class recurs or only the current instance is patched.

Use the **5 Whys** to trace the chain:

1. Why did the API return 500? → Database query threw an exception.
2. Why did the query throw? → It received a null parameter.
3. Why was the parameter null? → The upstream service omitted it.
4. Why did upstream omit it? → The field was renamed in a migration but not in the serializer.
5. Why wasn't the serializer updated? → Migration and serializer are in different repos with no shared contract.

The fix at level 1 (null check) addresses the surface failure. The fix at level 5 (shared contract + test) eliminates the bug class. Every time a cause seems found, ask one more "why."

---

## Reproduction as Understanding

If the bug cannot be triggered on demand, the triggering condition is still unknown.

- **Consistent reproduction** means the triggering conditions are identified.
- **Minimizing the reproduction** reveals what the bug depends on. If a 1000-line test triggers it, try a 10-line reproduction. Each removed piece that preserves the bug proves what the bug does NOT depend on. A truly small reproduction often makes the cause obvious.
- **Fast reproduction** enables rapid iteration. If each attempt takes 3 minutes, reduce reproduction time upfront. This pays for itself within a few cycles.
- **Confirmed intermittent failures** mean there's an uncontrolled variable — timing, ordering, external state. That variable IS the bug. Find it. If the failure is consistent across repeated runs, it is a deterministic bug — do not apply timing or async fixes.

---

## Stuck-Loop Traps

Use this table when debugging activity continues but the evidence set is not changing. Each row starts from an observable agent state, not from an inferred mindset.

| Observable signal | Failure mode | Recovery action |
|---|---|---|
| A hypothesis is named before the full error, actual values, or reproduction output are recorded | Theory before data | Record 3 concrete observations before naming a cause |
| New evidence is used only to support the current hypothesis | Confirmation bias | Name one observable result that would disprove the hypothesis, then run or inspect for it |
| The same hypothesis survives 2 failed experiments | Anchoring | Generate 3 competing explanations and choose the next observation that distinguishes them |
| The proposed fix targets the nearest stack-trace frame without ownership evidence | Proximity bias | Trace backward until a behavior-changing producer or caller is identified |
| The proposed cause matches a recent or familiar bug, but current evidence is not listed | Availability bias | List evidence from this failure before applying the familiar pattern |
| A value, config, state, fixture, or environment condition is assumed true but has not been observed | Assumption blindness | Print, inspect, query, or otherwise observe the actual value before editing |
| Patch attempts continue without recording what each attempt eliminated | Sunk-cost loop | Stop changing code; list eliminated possibilities, unresolved unknowns, and the next observation |
