# Mental Models — Full Detail

Extended explanations for the five mental models summarized in SKILL.md. Reach for this file when you need to apply a model deeply, have lost your bearings, or want to understand the reasoning behind a technique.

---

## Debugging Is a Search Problem

A bug is one broken thing among thousands of working things. Every action you take should eliminate possibilities. If an action doesn't shrink the search space, it's wasted.

The expert debugger asks: *"What's the fastest experiment I can run that eliminates the most remaining possibilities?"* — not *"What might be wrong?"*

When the search space is large, use **divide and conquer**: split it in half, test which half contains the bug, discard the other, repeat. This gives logarithmic convergence — even in a million-line codebase, ~20 well-chosen binary splits find the bug.

Ways to split:
- **Through code**: checkpoint halfway through execution. Data correct at midpoint? Bug is in the second half.
- **Through time**: `git bisect`. "Worked Tuesday, broken now" → bisect finds the exact breaking commit.
- **Through components**: in layered systems (UI → API → Service → DB), check data at each boundary. Bug lives between the last correct boundary and the first incorrect one.
- **Through input**: some inputs trigger it, others don't. Bisect the input space — which field, which value range?

---

## Observation Over Theory

*"Quit thinking and look."* — Agans

The natural impulse is to theorize immediately. Resist it. Observation first, theory second. Theories formed without data are usually wrong — and worse, they anchor you to a false explanation.

What observation looks like in practice:
- Read the **full** error message. Not just the first line — the type, the full stack trace, the first frame in *your* code, the data values. "What the error says" and "where in your code it originated" are different questions.
- **Print the actual value.** Don't assume you know what a variable is. The thing you're most sure about ("that can't be null here") is often exactly what's wrong.
- **Instrument at boundaries.** In multi-component systems, add logging at each boundary, run once, read where data breaks. One instrumented run costs less than three wrong hypotheses.
- **Find a working example** of the same pattern and diff it against the broken case. List every difference, however small.

---

## Symptom Site ≠ Owning Layer

The file or line where the error surfaces is where the symptom appears, not necessarily where the fix belongs. A stack trace points to the symptom site; the owning layer requires tracing the behavior back to its source.

When you see an error at `file.ts:42`, your first question is NOT "what's wrong at line 42?" — it's *"what put the system in a state where line 42 fails?"* The answer often lives in a completely different file, layer, or even repository.

Before accepting the stack-trace location as the bug, check all four competing explanations:
- **Expectation wrong?** Is the assertion itself testing for the wrong thing?
- **Fixture wrong?** Is the test setup producing bad input or state?
- **Environment wrong?** Is something outside the code (config, timing, deps) causing this?
- **Implementation wrong?** Is the production code actually at fault?

Check all four before committing to one.

---

## Causal Chain Depth

Every surface cause has a deeper cause. The depth at which you fix determines whether the bug class recurs or just this one instance.

Use the **5 Whys** to trace the chain:

1. Why did the API return 500? → Database query threw an exception.
2. Why did the query throw? → It received a null parameter.
3. Why was the parameter null? → The upstream service omitted it.
4. Why did upstream omit it? → The field was renamed in a migration but not in the serializer.
5. Why wasn't the serializer updated? → Migration and serializer are in different repos with no shared contract.

The fix at level 1 (null check) is a band-aid. The fix at level 5 (shared contract + test) eliminates the entire class of bug. Every time you think you've found the cause, ask one more "why."

---

## Reproduction as Understanding

If you can't make the bug happen on demand, you don't understand it yet.

- **Consistent reproduction** means you've identified the triggering conditions.
- **Minimizing the reproduction** reveals what the bug depends on. If a 1000-line test triggers it, can a 10-line test? Each piece you remove and the bug survives tells you what the bug does NOT depend on. A truly small reproduction often makes the cause obvious.
- **Fast reproduction** enables rapid iteration. If each attempt takes 3 minutes, invest time upfront to make it take 10 seconds. This pays for itself within a few cycles.
- **Intermittent failures** mean there's an uncontrolled variable — timing, ordering, external state. That variable IS the bug. Find it.
