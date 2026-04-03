# Response Language Reference

## Phrasing for Different Scenarios

**Correct feedback:** `"Fixed. [what changed]"` — or just fix it in code without comment.

**Pushback:** State the technical finding, reference the evidence, ask a specific question. Example: `"The current approach co-locates validation with the handler because callers depend on receiving domain-specific errors (see tests in X). Extracting to middleware would genericize the errors. Was that the intent, or were you aiming for something else?"`

**If you pushed back and were wrong:** `"You were right — I checked [X] and it does [Y]. Fixing."` No apology, no defense. State the correction and move on.

**If the reviewer misunderstood:** Don't say "you misunderstood." Instead: `"I think the disconnect is [X]. The code does [Y] because [Z]. I'll add a comment to make this clearer."` Fix the code clarity even when the suggestion is wrong — the confusion is real and will affect future readers.

**GitHub inline comments:** Reply in the comment thread, not as a top-level PR comment — `gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`
