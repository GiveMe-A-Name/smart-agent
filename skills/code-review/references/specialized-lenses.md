# Specialized Review Lenses

Apply these focused lenses when the change touches the relevant domain. They are depth adjustments for high-risk areas — not checklists to run on every review.

**Severity guidance:** Findings identified through Security, Migration, and Performance lenses start at **Important** by default — not Minor, regardless of how contained the finding looks in isolation. Downgrade only when the risk is blocked by a compensating control already in place and verifiable in the diff. [Because these lenses exist to surface risks in domains where the cost of a missed finding is highest — severity should reflect that domain risk, not just the local appearance of the code.]

---

## Security Review Lens

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

---

## Performance Review Lens

Apply when the change touches: database queries, API endpoints serving user traffic, data processing pipelines, anything on a hot path, or any code that scales with data volume or user count.

**Query and I/O patterns:**
- Does the change introduce N+1 query patterns (one query per item in a collection instead of a batch query)?
- Are database queries using appropriate indexes? A missing index on a WHERE clause column becomes a full table scan at scale.
- Is there I/O inside a loop that could be batched or parallelized?
- Are connections properly pooled and released? Leaked connections exhaust the pool under load.

**Scaling characteristics:**
- What is the algorithmic complexity? O(n²) in a test with 10 items passes; with 100,000 items in production it doesn't.
- Are collections unbounded? A list that grows with data volume without pagination or limits is a memory time bomb.
- Does the change degrade gracefully under load, or does it cliff (work fine until a threshold, then catastrophic failure)?

**Resource management:**
- Are large objects or streams properly closed/disposed after use?
- If caching is introduced, what is the eviction strategy? Unbounded caches are memory leaks with latency benefits.
- Are timeouts set on all external calls? A missing timeout on an HTTP call means one slow dependency can freeze the entire system.

---

## Migration and Data Safety Lens

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

---

## Dependency Review Lens

Apply when the change adds, removes, upgrades, or replaces an external dependency — including transitive dependency changes from version bumps.

**Before accepting a new dependency:**
- Is it actively maintained? Check: last commit date, open issue count, response time on issues, bus factor (single maintainer?).
- What is its security history? Check: known CVEs, security advisory history, whether maintainers respond to security reports.
- What is the blast radius? How much of your codebase will depend on this? A utility in one file is low-risk; a framework used everywhere is high-risk.
- What does it pull in transitively? A small library that depends on 50 other packages has a large supply-chain attack surface.
- Is there a simpler alternative? Standard library features, an existing dependency, or a small amount of custom code?
- License compatibility: is the dependency's license compatible with your project's license?

**For dependency upgrades:**
- Read the changelog between versions. Especially: breaking changes, deprecations, and security fixes.
- Check if the upgrade changes any transitive dependency that other parts of the system also depend on.

---

## API Design Review Lens

Apply when the change introduces or modifies public-facing or internal API endpoints, SDK methods, CLI commands, or any interface that other code or users will call.

**Consistency:**
- Does the new API follow existing conventions in this codebase? Naming patterns, parameter ordering, response shapes, error formats — inconsistency creates cognitive load for consumers.
- Are similar operations (create/read/update/delete) symmetric? If `GET /users/:id` returns `{ user: {...} }`, does `GET /users` return `{ users: [...] }` (not `[...]`)?

**Error contracts:**
- Are error responses structured and predictable? Consumers need to handle errors programmatically. A bare string error is less useful than `{ error: { code: "NOT_FOUND", message: "..." } }`.
- Are all error cases documented or at minimum discoverable from the code?

**Idempotency and safety:**
- Can this operation be safely retried? For operations that modify state, is there an idempotency mechanism (idempotency key, natural idempotency through PUT semantics)?
- Does the API distinguish between safe-to-retry operations (GET, PUT, DELETE by ID) and those that are not (POST that creates a resource)?

**Pagination and limits:**
- If the endpoint returns a collection, is there pagination? An unbounded list endpoint is a production incident at scale.
- Are there rate limits or request size limits? An endpoint that accepts arbitrary-size payloads is a DoS vector.
