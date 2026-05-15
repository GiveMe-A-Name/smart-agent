# Response Language Reference

## Phrasing for Different Scenarios

**Correct feedback:** `"Fixed. [what changed]"` — or just fix it in code without comment.

**Pushback:** State the technical finding, reference the evidence, ask a specific question. Example: `"The current approach co-locates validation with the handler because callers depend on receiving domain-specific errors (see tests in X). Extracting to middleware would genericize the errors. Was that the intent, or were you aiming for something else?"`

**Clarification request:** Ask a specific question, not an open-ended one. The question should name the two interpretations and state what changes based on the answer. Example: `"Are you suggesting we validate inputs at the middleware layer (which changes the error contract for all callers), or just add validation before the specific database call at line 42? The two approaches have different tradeoffs and I want to implement the right one."` — Not: `"Can you clarify what you mean?"` A clarification request that doesn't name what you need to evaluate gives the reviewer no information about how to help.

**If you pushed back and were wrong:** `"Confirmed: I checked [X] and it does [Y]. Fixing."` No apology, no defense, no performative agreement. State the correction and move on.

**If the reviewer misunderstood:** Don't say "you misunderstood." Instead: `"I think the disconnect is [X]. The code does [Y] because [Z]. I'll add a comment to make this clearer."` Fix the code clarity even when the suggestion is wrong — the confusion is real and will affect future readers.

**GitHub inline comments:** Reply in the comment thread, not as a top-level PR comment — `gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`
