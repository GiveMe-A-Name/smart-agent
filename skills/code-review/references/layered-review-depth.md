# Layered Review Depth

Deep guidance for each layer in the code review model. Reach for this when a layer needs more than the compact table in SKILL.md.

---

## Layer 0: Change Justification — "Should this change exist?"

This is the question junior reviewers rarely ask and senior reviewers ask first. Before reading a single line of implementation, establish:

**Read the PR description, linked spec, commit messages, or plan.** A reviewer who does not know what the change is supposed to accomplish cannot tell whether the implementation is correct — only whether it compiles.

Then ask:

- **What problem does this change solve?** Can you state it in one sentence? If not, the change may lack focus or the description is insufficient. Ask for clarification before proceeding.
- **Does this problem actually need solving right now?** Is there evidence (user reports, metrics, production incidents, product requirements) that this is worth the complexity it introduces? "Might be useful someday" is not justification.
- **Is the direction consistent with where the system is headed?** If the team is migrating away from FooWidget, a change that deepens investment in FooWidget is moving backward — no matter how well-written the code is. Check if the change aligns with known migrations, deprecations, or architectural decisions.
- **Is the scope right?** Does the change do exactly one thing, or does it bundle unrelated modifications that should be separate? A change that mixes a refactor, a feature, and a config tweak is harder to review, harder to revert, and harder to understand in git history.

**If the change should not exist**, stop here. Surface the concern with a constructive alternative. Reviewing implementation details of a change that shouldn't land wastes everyone's time.

**Common failure mode — skipping Layer 0 because the PR description makes the purpose clear.** Reading the description tells you what the author intended. It does not tell you whether the code achieves it, or whether the goal itself is worth achieving now. Layer 0 requires active judgment on the *change*, not passive acceptance of the stated purpose. A reviewer who skips Layer 0 can produce a thorough Layer 2-3 review of a change that should have been rejected at the first question.

---

## Layer 1: Design and Approach — "Is this the right solution?"

The change addresses a real problem. Now: is this the right way to solve it?

- **Is there a simpler approach?** The best code is code that doesn't need to exist. Could this be solved with a configuration change, an existing library, or by removing code? If a 200-line change can be replaced by 20 lines using an existing abstraction, that matters more than whether the 200 lines are well-written.
- **Does the overall structure make sense?** Do the interactions between pieces of code in the change make sense? Does the new code integrate well with the existing system?
- **Is it over-engineered?** Abstractions, parameters, extension points, or generic patterns with no current callers. "Supports future testability" or "could be useful later" are not justification — solve the future problem when its actual shape is visible.
- **Is it under-engineered?** Shortcuts that will force a rewrite soon: a hardcoded list that should be a config, a synchronous call that will block under load, a missing abstraction that forces copy-paste in the next feature.
- **What are the second-order effects?** A new function signature changes the contract for all callers. A new database column affects migration scripts, ORMs, and downstream consumers. A new dependency adds to build time and supply-chain risk. Trace the blast radius.
- **Does this change improve or degrade the system's overall health?** A change that adds complexity, reduces test coverage, or introduces inconsistency is degrading system health — even if it "works." Don't accept changes that make the codebase worse.

---

## Layer 2: Implementation Correctness — "Does the code do what it claims?"

- **Correctness.** Trace the logic path for normal cases, then for edge cases. Off-by-one errors, null/undefined handling, incorrect boolean logic, missing return statements.
- **Caller and usage impact.** If a function signature, return type, error behavior, or side effect changed, trace to every caller. Changes that look self-contained often break at the call site. This is one of the most commonly missed categories.
- **Error handling.** Errors caught at the right layer? Failures producing useful signals or silent data corruption? Catch blocks that swallow exceptions and return a default that hides the problem?
- **Security.** Injection vectors (SQL, XSS, command injection), unvalidated inputs, exposed secrets, privilege escalation, unsafe deserialization, TOCTOU race conditions. If the change touches auth or user input, increase scrutiny — see `specialized-lenses.md`.
- **Concurrency.** Shared state, threads, async operations, distributed coordination: race conditions, deadlocks, ordering assumptions. These require careful reading — you cannot catch them by running the code.
- **Consistency with codebase patterns.** Similar patterns elsewhere that should have been updated in the same change? A partial fix that leaves the same bug in three other places is not a fix.

---

## Layer 3: Maintainability and Tests — "Will this hold up over time?"

- **Complexity.** Can you understand what each function does without re-reading it? Long functions, deep nesting, clever tricks, and implicit state all contribute to future bugs.
- **Naming.** A good name eliminates the need for a comment. A bad name actively misleads.
- **Comments.** Useful comments explain *why*, not *what*. Code should be self-explanatory for the what. Comments about non-obvious business rules, workarounds for external system bugs, or "why not the obvious approach" are valuable.
- **Test quality.** Tests exist — but are they worth having?
  - Do they test real behavior or just exercise mocks?
  - Will they fail when the code breaks, or are assertions so loose they pass regardless?
  - Do they cover edge cases and error paths, or only the happy path?
  - Does a failing test tell you what went wrong without debugging?
  - Tests are also code to maintain — don't accept unnecessary complexity in tests.
- **Documentation.** If the change affects how users build, test, or interact with the system, does it update relevant documentation? If code is deleted or deprecated, is that reflected in docs?

---

## Review Navigation

Don't read every diff hunk top-to-bottom. Navigate deliberately.

**Step 1 (Layer 0):** Read the PR description, linked issues, commit messages. Restate the goal in your own words. If you can't, ask for clarification before reviewing code. Scan the file list — is it focused? A change touching 30 files across 5 subsystems is a red flag for scope creep. **If the change shouldn't exist, stop and say so.**

**Write your Layer 0 conclusion before opening any implementation file.** This step is a gate, not a warmup: if you have not recorded a Layer 0 assessment — even "direction is correct, scope is appropriate" — you have not completed Step 1 and should not proceed. Layer 0 conclusions formed after reading implementation code are anchored to what the code does, not whether the direction is right; that anchoring is exactly the bias this step exists to counteract.

**Step 2 (Layer 1):** Find the files that represent the "main" part of the change — usually the file with the most logical changes, not the most lines. Read the core design first. **Surface major design problems immediately** — the author may be building more work on top of this.

**Write your Layer 1 conclusion before recording Layer 2-3 findings.** If the design has a fundamental problem, that problem is the review — continuing to catalog implementation details signals implicit approval of the approach.

**Step 3 (Layers 2–3):** Once design is confirmed sound, review remaining files in logical order. For each changed function, read enough context to understand what the code does in the broader system.

**For remote PRs:** Read the PR description and any existing review comments before starting.

**For local changes:** `git diff` for unstaged, `git diff --staged` for staged.
