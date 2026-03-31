---
name: code-review
description: "Invoke when code changes exist — local diff, staged changes, or a remote PR — and the task is to evaluate them. Cost of unnecessary invocation: a short review pass. Cost of missing: bugs in production, or false safety from an incomplete review."
---

# Code Review

A review that catches nothing is worse than no review — it creates false confidence. But a review that only catches typos while missing a flawed design is barely better. The highest-value findings are not bugs in the implementation — they are problems with the change itself: wrong direction, wrong abstraction, wrong scope.

## Trigger Logic

**Invocation default**: Skipping structured review before merge lets problems compound. The cost of unnecessary rigor is a short overhead. The cost of missing it is bugs in production or findings that block merge after the fact.

Use when:
- code changes exist (local diff, staged changes, or remote PR) and the task is to evaluate them
- completing a task in subagent-driven development and review is the gate before the next step
- user asks to "review", "check", or "audit" code changes

Do not use when:
- review feedback already exists and the next step is deciding what to implement, challenge, or clarify — that is `receiving-code-review`, not a code review
- no diff or changed code exists to evaluate yet — if you are writing code, invoke this skill after the code exists

## Capability Boundary

This skill owns: reading code changes, identifying issues across all layers (justification through implementation), calibrating severity, and delivering a structured verdict.

It does NOT:
- implement fixes or refactor reviewed code
- verify that previously given feedback was applied — that belongs to the next review cycle
- claim coverage over code that was not read

## Invariants

**Upper-layer problems outweigh lower-layer problems.** A change that goes in the wrong direction does not become acceptable because the implementation is clean. A design that introduces unnecessary coupling is not redeemed by thorough tests. Always work from the top layer down — if a higher-layer problem is severe enough, stop and surface it immediately rather than cataloging line-level issues in code that may need to be rewritten.

**Read context, not just diff hunks.** For every changed function or module, read enough surrounding code to understand: what callers expect, what invariants the module maintains, and what the changed lines actually do in context. A diff line that looks correct in isolation often reveals a problem when read against the code that calls it.

**Only give feedback on code you actually read.** Never approve or comment on code outside the diff or files explicitly opened. "Looks good" without reading is a false safety signal.

**Every issue needs: file:line, what is wrong, why it matters.** Vague findings ("improve error handling", "this could be cleaner") are not actionable.

**Severity must reflect actual risk.** Critical means data loss, security vulnerability, broken functionality, or blocking regression — not style, not preference. Inflating severity makes all feedback less trustworthy.

**Give a clear verdict.** Every review ends with: Ready / Not ready / Ready with fixes. An ambiguous conclusion defers the merge decision to the author, which is the reviewer's job.

**Before exiting this skill, you MUST complete the Self-Check section at the end.**

---

## The Layered Review Model

Expert reviewers think in layers, from the most consequential questions down to the least. This is the core of the review — not a checklist to scan, but a sequence of increasingly focused lenses applied to the change.

**The rule: a problem at a higher layer is almost always more important than a problem at a lower layer.** Wrong direction with perfect code is still wrong. Right direction with a few bugs is fixable.

### Layer 0: Change Justification — "Should this change exist?"

This is the question junior reviewers rarely ask and senior reviewers ask first. Before reading a single line of implementation, establish:

**Read the PR description, linked spec, commit messages, or plan.** A reviewer who does not know what the change is supposed to accomplish cannot tell whether the implementation is correct — only whether it compiles.

Then ask:

- **What problem does this change solve?** Can you state it in one sentence? If not, the change may lack focus or the description is insufficient. Ask the author for clarification before proceeding.
- **Does this problem actually need solving right now?** Is there evidence (user reports, metrics, production incidents, product requirements) that this is worth the complexity it introduces? "Might be useful someday" is not justification.
- **Is the direction consistent with where the system is headed?** If the team is migrating away from FooWidget, a change that deepens investment in FooWidget is moving backward — no matter how well-written the code is. Check if the change aligns with known migrations, deprecations, or architectural decisions.
- **Is the scope right?** Does the change do exactly one thing, or does it bundle unrelated modifications that should be separate? A change that mixes a refactor, a feature, and a config tweak is harder to review, harder to revert, and harder to understand in git history.

**If the change should not exist**, stop here. Surface the concern immediately with a constructive alternative ("This area is being deprecated — consider building on BarWidget instead"). Reviewing implementation details of a change that shouldn't land wastes everyone's time.

### Layer 1: Design and Approach — "Is this the right solution?"

The change addresses a real problem. Now: is this the right way to solve it?

- **Is there a simpler approach?** The best code is code that doesn't need to exist. Could this be solved with a configuration change, an existing library, or by removing code rather than adding it? If a 200-line change can be replaced by 20 lines using an existing abstraction, that matters more than whether the 200 lines are well-written.
- **Does the overall structure make sense?** Do the interactions between the pieces of code in the change make sense? Does the new code integrate well with the existing system, or does it sit awkwardly alongside it?
- **Is it over-engineered?** Are there abstractions, parameters, extension points, or generic patterns that have no current callers? "Supports future testability" or "could be useful later" are not justification — the future problem should be solved when it arrives and its actual shape is visible. Flag code that solves speculative problems.
- **Is it under-engineered?** Conversely, does the solution take shortcuts that will force a rewrite soon? A hardcoded list that should be a configuration, a synchronous call that will block under load, a missing abstraction that forces copy-paste in the next feature.
- **What are the second-order effects?** Changes rarely exist in isolation. A new function signature changes the contract for all callers. A new database column affects migration scripts, ORMs, and downstream consumers. A new dependency adds to build time and supply-chain risk. Trace the blast radius.
- **Does this change improve or degrade the system's overall health?** Most systems become complex through many small changes that each seem reasonable in isolation. A change that adds complexity, reduces test coverage, or introduces inconsistency is degrading system health — even if it "works." Don't accept changes that make the codebase worse, even slightly. Hold the line.

### Layer 2: Implementation Correctness — "Does the code do what it claims?"

The design is sound. Now verify the implementation.

- **Correctness.** Does the code actually achieve its stated purpose? Trace the logic path for normal cases, then for edge cases. Look for off-by-one errors, null/undefined handling, incorrect boolean logic, and missing return statements.
- **Caller and usage impact.** If a function signature, return type, error behavior, or side effect changed, trace to every caller. Changes that look self-contained often break at the call site. This is one of the most commonly missed categories in reviews.
- **Error handling.** Are errors caught at the right layer? Do failures produce useful signals (logs, metrics, user-facing messages) or silent data corruption? Is there a catch block that swallows exceptions and returns a default that hides the problem?
- **Security.** Injection vectors (SQL, XSS, command injection), unvalidated inputs, exposed secrets, privilege escalation, unsafe deserialization, TOCTOU race conditions. If the change touches authentication, authorization, or user input, increase scrutiny.
- **Concurrency.** If the code involves shared state, threads, async operations, or distributed coordination: are there race conditions, deadlocks, or ordering assumptions? These are nearly impossible to catch by running the code — they require careful reading.
- **Consistency with codebase patterns.** Are there similar patterns elsewhere in the codebase that should have been updated in the same change? A partial fix that leaves the same bug in three other places is not a fix. Does the change follow existing conventions for error handling, logging, naming?

### Layer 3: Maintainability and Tests — "Will this hold up over time?"

The code works today. Will it be understandable and modifiable six months from now by someone who didn't write it?

- **Complexity.** Can you understand what each function does without re-reading it three times? "Too complex" means "developers are likely to introduce bugs when they try to modify this code." Long functions, deep nesting, clever tricks, and implicit state all contribute.
- **Naming.** Do names communicate what things are and what they do? A good name eliminates the need for a comment. A bad name actively misleads.
- **Comments.** Useful comments explain *why*, not *what*. If a comment explains what the code does, the code should be rewritten to be self-explanatory. But comments about non-obvious business rules, workarounds for external system bugs, or "why not the obvious approach" are valuable.
- **Test quality.** Tests exist — but are they worth having?
  - Do they test real behavior or just exercise mocks?
  - Will they actually fail when the code breaks, or are they so loosely asserted that they pass regardless?
  - Do they cover the important edge cases and error paths, or only the happy path?
  - Are tests clear enough that a failing test tells you what went wrong without debugging?
  - Tests are also code that has to be maintained — don't accept unnecessary complexity in tests.
- **Documentation.** If the change affects how users build, test, or interact with the system, does it update the relevant documentation? If code is deleted or deprecated, is that reflected in docs?

---

## Review Procedure

Don't just read every diff hunk top-to-bottom. Navigate the change deliberately.

### Step 1: Understand the change (Layer 0)

Read the PR description, linked issues, commit messages. Restate the goal in your own words. If you can't, the description is insufficient — ask for clarification before reviewing code.

Scan the file list and overall shape of the change. Is it focused? Is the size reasonable? A change touching 30 files across 5 subsystems is a red flag for scope creep.

**If the change shouldn't exist at all, stop and say so.** Don't review implementation details of a misguided change.

### Step 2: Evaluate the core design (Layer 1)

Find the files that represent the "main" part of the change — usually the file with the most logical changes, not the most lines. Read the core design first. This gives you context for all the supporting changes.

**If there are major design problems, surface them immediately** even if you haven't finished reviewing the rest. The author may be building more work on top of this design. The sooner design issues are raised, the less work is wasted.

### Step 3: Review the implementation (Layers 2-3)

Once the design is confirmed sound, review the remaining files in a logical order. For each changed function, read enough context to understand what the code does in the broader system.

### Calibrating review depth

Not all changes deserve the same scrutiny. Adjust depth based on risk:

| Change characteristics | Depth |
|---|---|
| Touches auth, payments, data deletion, crypto, or user-facing security | Maximum — read every line, trace every path |
| New feature with significant logic, new API surface, or schema migration | High — full Layer 0-3 review |
| Bug fix with clear scope and existing test coverage | Medium — verify fix correctness, check for similar bugs elsewhere, verify test |
| Config change, dependency bump, documentation update | Light — verify intent, check for obvious errors |
| Generated code, formatting-only changes, trivial renames | Scan — verify the change is what it claims to be |

---

## Severity Calibration

| Severity | What qualifies | Default action |
|---|---|---|
| Critical | Data loss, security hole, broken existing functionality, blocking regression, wrong direction (Layer 0/1 problems that invalidate the change) | Must fix before merge |
| Important | Missing error handling, test gaps, unmet requirements, design concerns that don't invalidate but weaken the change, unaddressed caller impact | Should fix; defer only with written justification |
| Minor | Style, naming, optimization, documentation improvements, subjective preferences | Author decides |

**Layer 0/1 problems are almost always Critical or Important.** A change that degrades system health or solves the wrong problem is not a Minor issue.

---

## Remote PR vs Local Changes

- For remote PRs: read the PR description and any existing review comments before starting. Understanding intent prevents misread findings.
- For local changes: `git diff` for unstaged, `git diff --staged` for staged. Clarify scope with the author if it is ambiguous.

## Automated Checks

Run the project's preflight, lint, or test suite when the change is substantial or when test coverage is in question. A passing review with failing automated checks is not a passing review. Skip only when infeasible (no test runner configured, environment not available) — if skipped, say so explicitly in the review.

---

## Specialized Review Lenses

Apply these focused lenses when the change touches the relevant domain. They are not checklists to run on every review — they are depth adjustments for high-risk areas.

### Security Review Lens

Apply when the change touches: authentication, authorization, user input handling, data storage, external API integration, cryptography, file uploads, or any code that runs with elevated privileges.

**Input and injection:**
- Is user input validated at the boundary before being used? Look for SQL injection (string concatenation in queries), XSS (unescaped output in HTML), command injection (user data in shell commands), path traversal (user data in file paths).
- Are parameterized queries used for all database access? String interpolation in SQL is never acceptable.
- If the change parses structured input (JSON, XML, YAML), is there protection against oversized payloads, deeply nested structures, or entity expansion attacks?

**Authentication and authorization:**
- If a new endpoint or data path is added, are both authentication (who are you?) and authorization (are you allowed to do this?) enforced?
- Is authorization checked at the service/API layer, not just the UI? A missing server-side check means the UI is the only barrier.
- Are there IDOR (Insecure Direct Object Reference) risks? Can user A access user B's data by guessing or incrementing an ID?

**Data exposure:**
- Are sensitive fields (passwords, tokens, PII) excluded from logs, error messages, and API responses?
- If the change adds logging, does it avoid logging request bodies, headers with tokens, or other sensitive data?
- Are secrets (API keys, passwords, certificates) stored in environment variables or a secrets manager — never in code, config files committed to git, or comments?

**Dependencies:**
- If a new dependency is added: is it actively maintained? Has it had known security vulnerabilities? What is the blast radius if it is compromised?

### Performance Review Lens

Apply when the change touches: database queries, API endpoints serving user traffic, data processing pipelines, anything on a hot path, or any code that scales with data volume or user count.

**Query and I/O patterns:**
- Does the change introduce N+1 query patterns (one query per item in a collection instead of a batch query)?
- Are database queries using appropriate indexes? A missing index on a WHERE clause column becomes a full table scan at scale.
- Is there I/O inside a loop that could be batched or parallelized?
- Are connections properly pooled and released? Leaked connections exhaust the pool under load.

**Scaling characteristics:**
- What is the algorithmic complexity? O(n^2) in a test with 10 items passes; with 100,000 items in production it doesn't.
- Are collections unbounded? A list that grows with data volume without pagination or limits is a memory time bomb.
- Does the change degrade gracefully under load, or does it cliff (work fine until a threshold, then catastrophic failure)?

**Resource management:**
- Are large objects or streams properly closed/disposed after use?
- If caching is introduced, what is the eviction strategy? Unbounded caches are memory leaks with latency benefits.
- Are timeouts set on all external calls? A missing timeout on an HTTP call means one slow dependency can freeze the entire system.

### Migration and Data Safety Lens

**Perspective**: This lens evaluates migration and data safety from the **reviewer's perspective** — will this change break production during or after deployment? The reviewer checks whether the migration plan is safe and whether the author considered backward compatibility. Whether the migration can actually be operated safely is an operability concern; how the migration logic should be implemented is an implementation judgment concern.

Apply when the change includes: database schema changes, data migrations, API response shape changes, configuration format changes, feature flag changes, or anything that requires coordinated deployment across services.

**Schema migrations:**
- Is the migration backward-compatible with the currently deployed code? A migration that renames or removes a column while old code is still reading it will cause production errors during the deployment window.
- Safe migration pattern: add new column → deploy code that writes to both → backfill → deploy code that reads from new → remove old column. A single migration that does add+remove is a red flag.
- Does the migration have a rollback path? If the deployment fails, can the migration be reversed without data loss?
- For large tables: will the migration lock the table? What's the estimated runtime? Does it need to run in batches?

**API contract changes:**
- Does this change remove, rename, or change the type of any field in an API response? Even "internal" APIs may have consumers you don't know about — scripts, cron jobs, partner integrations, mobile clients on old versions.
- If the change modifies error response formats, check whether any caller parses error responses (many do, even when they shouldn't).
- Versioning: is there a versioning strategy? If a breaking change is unavoidable, is there a deprecation path?

**Configuration and deployment:**
- Does the change rename environment variables, change default values, or alter config file structure? These break silently — the application starts, but behaves differently.
- If the change requires "deploy A before B" ordering, is this documented and enforceable?
- Feature flags: if used, is there a plan to remove the flag after rollout? Permanent feature flags are tech debt.

### Dependency Review Lens

Apply when the change adds, removes, upgrades, or replaces an external dependency — including transitive dependency changes from version bumps.

**Before accepting a new dependency:**
- Is it actively maintained? Check: last commit date, open issue count, response time on issues, bus factor (single maintainer?).
- What is its security history? Check: known CVEs, security advisory history, whether the maintainers respond to security reports.
- What is the blast radius? How much of your codebase will depend on this? A utility used in one file is low-risk; a framework used everywhere is high-risk.
- What does it pull in transitively? A small library that depends on 50 other packages has a large supply-chain attack surface.
- Is there a simpler alternative? Could this be achieved with standard library features, an existing dependency, or a small amount of custom code?
- License compatibility: is the dependency's license compatible with your project's license?

**For dependency upgrades:**
- Read the changelog between versions. Especially: breaking changes, deprecations, and security fixes.
- Check if the upgrade changes any transitive dependency that other parts of the system also depend on (version conflicts).

### API Design Review Lens

Apply when the change introduces or modifies public-facing or internal API endpoints, SDK methods, CLI commands, or any interface that other code or users will call.

**Consistency:**
- Does the new API follow the existing conventions in this codebase? Naming patterns, parameter ordering, response shapes, error formats — inconsistency across similar endpoints creates cognitive load for consumers.
- Are similar operations (create/read/update/delete) symmetric? If `GET /users/:id` returns `{ user: {...} }`, does `GET /users` return `{ users: [...] }` (not `[...]`)?

**Error contracts:**
- Are error responses structured and predictable? Consumers need to handle errors programmatically. A bare string error is less useful than `{ error: { code: "NOT_FOUND", message: "..." } }`.
- Are all error cases documented or at minimum discoverable from the code? Can a consumer tell what errors to expect without reading the implementation?

**Idempotency and safety:**
- Can this operation be safely retried? For operations that modify state, is there an idempotency mechanism (idempotency key, natural idempotency through PUT semantics)?
- Does the API distinguish between operations that are safe to retry (GET, PUT, DELETE by ID) and those that are not (POST that creates a resource)?

**Pagination and limits:**
- If the endpoint returns a collection, is there pagination? An unbounded list endpoint is a production incident at scale.
- Are there rate limits or request size limits? An endpoint that accepts arbitrary-size payloads is a DoS vector.

---

## Failure Signals

Stop and reassess if:

- you are reviewing implementation details before understanding what the change is supposed to do — go back to Layer 0
- you have not asked "should this change exist?" and "is this the right approach?" — you skipped the highest-value layers
- you are reviewing diff hunks without having read the surrounding context for each changed function
- you are about to write "looks good" or approve without having read the code
- every finding is labeled Critical — severity inflation makes all feedback untrustworthy
- all your findings are Layer 3 (style, naming, minor cleanups) and you found nothing at Layers 0-2 — either the change is genuinely excellent, or you didn't look hard enough at the higher layers
- findings lack file:line references — specificity is the baseline for actionable feedback
- the review ends without a verdict — an open-ended summary is not a review
- you are giving feedback on code outside the diff — you are responsible for what you reviewed, not what you assumed
- a finding describes a vague risk without naming what breaks and why ("this could cause issues")
- you assessed correctness without checking the callers of changed functions — the call site is often where the real problem surfaces
- the change includes a schema migration, API change, or config change and you did not apply the Migration and Data Safety lens
- you accepted a new dependency without checking its maintenance status, security history, and license


## Reviewing AI-Generated Code

AI-generated code requires the same review rigor as human-written code. The layered review model and all specialized lenses apply unchanged. This section covers reviewer-specific adjustments — what to look for differently when you know or suspect the code was AI-generated.

**Verify external references against source.** AI tools generate plausible-but-nonexistent APIs, incorrect method signatures, and outdated library interfaces. As a reviewer, you cannot trust that imports and API calls were validated by the author. Spot-check external API calls, especially for less common libraries — open the actual docs or source code rather than relying on your own familiarity.

**Increase skepticism of "looks right" code.** AI-generated code passes superficial reading more easily than human code — correct structure, reasonable names, sensible-looking flow — while containing subtle logic errors. Slow down at boundary conditions, off-by-one scenarios, and null/undefined handling. If the code looks too clean for its complexity, trace the edge cases manually.

**Search for duplicated functionality.** AI tools lack full codebase awareness and frequently generate functions that already exist elsewhere. Before accepting a new utility, helper, or abstraction, search the codebase for existing equivalents. This is a reviewer responsibility because the author (especially an AI-assisted one) may not have checked.

**Scrutinize test assertions.** AI-generated tests frequently mock the system under test, assert on implementation details, or pass regardless of behavior changes. Check: would this test fail if the behavior actually broke? Tests that exercise mocks instead of real behavior provide false coverage.

---

## Output Format

See `templates/review-output.md` for the full output structure.

**Required sections:** Change Assessment (Layer 0-1 findings) · Strengths · Issues (by severity) · Verdict

**For each issue:**
- Layer reference (which layer this finding belongs to)
- `File:line` reference
- What is wrong
- Why it matters
- How to fix (if not obvious)

**Verdict options:** `Ready to merge` / `Not ready` / `Ready with fixes`

---

## Self-Check Before Exiting

- [ ] Did I understand intent and evaluate Layers 0-1 (should this exist? right approach?) before reviewing implementation?
- [ ] Did I read enough context around each changed function — including callers — not just the diff hunks?
- [ ] Does every issue have file:line, what is wrong, why it matters — with honest severity calibration?
- [ ] Did I apply relevant specialized lenses (Security, Performance, Migration, Dependency, API Design)?
- [ ] If AI-generated code: did I verify external API references against source, search for duplicated functionality, and scrutinize test assertions?
- [ ] Did I give a clear verdict (Ready / Not ready / Ready with fixes)?
- [ ] Am I exiting because the review is genuinely complete, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
