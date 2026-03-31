---
name: security-thinking
description: "Invoke when the code being written or reviewed handles untrusted input, manages authentication or authorization, stores or transmits sensitive data, adds new dependencies, or exposes new interfaces. Cost of unnecessary invocation: a short security review pass. Cost of missing: a vulnerability that costs orders of magnitude more to fix after deployment than before. When uncertain whether security considerations apply, invoke."
---

# Security Thinking

Think about security as a design dimension, not an afterthought. Security vulnerabilities are not exotic edge cases — they are the natural consequence of code that trusts data it shouldn't, exposes more than it needs to, or grants more privilege than required.

This skill teaches the judgment to recognize security-relevant decisions during design and implementation — before they become vulnerabilities.

## Trigger Logic

**Invocation default**: A security flaw caught before deployment costs a code change. The same flaw after deployment costs an incident, a disclosure, user trust, and possibly regulatory consequences. The cost asymmetry is extreme. Invoke when any security-relevant dimension is present in the work.

Use when:
- code handles untrusted input (user input, API responses from external services, file uploads, URL parameters, headers)
- code manages authentication, authorization, or session handling
- code stores, transmits, or logs data that could be sensitive (PII, credentials, tokens, financial data)
- a new dependency is being added to the project
- a new endpoint, interface, or data path is being exposed
- code involves cryptography, hashing, or token generation

Do not use when:
- the change is confirmed to be internal tooling with no external input and no access to sensitive data
- the change is pure documentation or comments with no code effect

## Capability Boundary

This skill owns:
- identifying security-relevant aspects of code changes
- applying threat modeling thinking to design and implementation decisions
- judging the security posture impact of architectural choices
- recognizing when code is making implicit trust decisions that should be explicit

This skill does not own:
- penetration testing or security tool configuration
- compliance frameworks (SOC 2, HIPAA, PCI) — though security thinking often satisfies compliance
- incident response after a breach
- security infrastructure (WAFs, IDS) architecture

## Invariants

- All data from outside the trust boundary is untrusted until validated at the boundary.
- Authentication (who are you?) and authorization (are you allowed?) are separate concerns and must both be present.
- Security through obscurity is not security — assume attackers have the source code.
- The principle of least privilege applies to every layer: code, APIs, database users, service accounts, deployment permissions.
- Sensitive data must be protected at rest, in transit, and in logs.
- **Before exiting this skill, you MUST complete the Self-Check section at the end.**

---

## Reading the Situation

Before working through the judgment dimensions, identify which ones matter for the code you are looking at. Not every change needs all five — but picking the wrong subset is how vulnerabilities slip through.

| What you see in the code | Focus on |
|---|---|
| New endpoint or route added | Trust Boundaries + Auth |
| User input flows into a query, command, or template | Trust Boundaries |
| API response from external service consumed | Trust Boundaries |
| Login, signup, password reset, or session logic | Auth |
| Resource accessed by ID from request parameters | Auth (IDOR specifically) |
| New middleware, decorator, or guard added | Auth |
| Error messages returned to the user | Data Exposure |
| Logging statements added or modified | Data Exposure |
| API response shape defined or changed | Data Exposure |
| New `npm install`, `pip install`, or dependency added | Dependency Trust |
| Dependency version updated or unpinned | Dependency Trust |
| API keys, tokens, or connection strings in code | Secrets |
| Hashing, encryption, or token generation logic | Secrets |
| File upload or download handling | Trust Boundaries + Data Exposure |
| Webhook or callback URL processing | Trust Boundaries + Auth |

If the code change doesn't match any row, ask: *does this code touch data that crosses a boundary, or make a decision about who can do what?* If yes, at least one dimension applies. If genuinely no, this skill may not be needed for this change.

---

## Judgment Dimensions

### 1. Trust Boundaries

**Question**: Where does untrusted data enter this code, and what happens to it before it's used?

**Key signals**:
- **Data flow from boundary to use**: Trace user input from where it arrives to where it's consumed. Every step between entry and use that lacks validation is attack surface. The boundary is the last point where you control what the data looks like.
- **Validation placement**: Validation must happen at the trust boundary — not deeper in the call stack where "someone should have already checked." If you cannot point to the exact line where untrusted input is validated before use, it isn't validated.
- **Injection vectors**: When untrusted data is inserted into a structured context (SQL, HTML, shell command, file path, URL, template), the question is always: *can the attacker break out of the data position and into a control position?* String concatenation into any structured language is the primary signal. Parameterized queries, auto-escaping template engines, and allowlists are the fix — not denylists or manual escaping.
- **Deserialization as code execution**: Formats that can execute code during deserialization (Python `pickle`, Java serialization, YAML `unsafe_load`) turn data channels into code execution channels. If untrusted data hits these parsers, it's equivalent to `eval()`.
- **Implicit trust**: Watch for code that trusts HTTP headers (`X-Forwarded-For`, `Referer`), client-side state, URL parameters, or data from "internal" APIs without re-validation. "Internal" services get compromised. Headers get spoofed.

### 2. Authentication and Authorization

**Question**: Who can reach this code path, and should they be able to?

**Key signals**:
- **Missing auth check**: A new endpoint or route without an authentication check is open to the world by default. Look for the auth middleware, decorator, or guard — if it's not there, it's not protecting the route. Relying on the URL being "hard to guess" is not authentication.
- **Auth vs. authz conflation**: Code that checks "is the user logged in?" but not "is this user allowed to do this specific thing?" has authentication without authorization. These are separate gates and both must be present.
- **IDOR (Insecure Direct Object Reference)**: When a resource is accessed by ID from request parameters (`GET /api/users/123/docs/456`), the question is: *does the code verify that the requesting user has access to this specific resource, or does it only verify they're authenticated?* Checking that a user is logged in does not mean they can access any arbitrary resource by guessing IDs.
- **Privilege decisions in the wrong layer**: Authorization checks in the UI (hiding buttons, filtering menus) without corresponding server-side checks are cosmetic, not security. An attacker doesn't use your UI. Check authorization at the API/service layer.
- **Session and token handling**: Sessions must use cryptographically random tokens, be stored securely (HttpOnly, Secure, SameSite cookies), and have reasonable expiration. Custom auth implementations are almost always worse than established libraries (OAuth, OIDC providers, framework auth modules) — the question to ask is: *why are we not using the standard solution?*
- **Rate limiting and lockout**: Login endpoints, password reset flows, and token exchange endpoints without rate limiting are vulnerable to brute force. If the code handles authentication events, look for throttling.

### 3. Data Exposure

**Question**: What sensitive data does this code touch, and where could it leak?

**Key signals**:
- **Log statements with request bodies**: Logging request or response bodies without filtering is the most common path to sensitive data in logs. PII, tokens, passwords, and credit card numbers end up in log aggregators that have broader access than the application itself. The fix is a sanitization filter applied at the logging layer — not developer discipline to "remember" what not to log.
- **Error messages to users**: Detailed error information (stack traces, SQL errors, internal IDs, file paths) returned in API responses tells attackers about your internals. Production error responses must be generic; detailed errors go to server-side logs.
- **API response over-exposure**: Returning full database objects when the client only needs three fields exposes every field — including ones added later that might be sensitive. Define explicit response shapes. The question is: *does this response contain only what the client needs, or everything the database has?*
- **Sensitive data at rest**: Passwords must be hashed with bcrypt, scrypt, or argon2 — never fast hashes (MD5, SHA1, SHA256). The required property for password storage is being slow. Other sensitive data (PII, financial data) should be encrypted at the database or application level.
- **Data in transit**: TLS everywhere, including "internal" service-to-service traffic. Internal networks get compromised, and unencrypted internal traffic is a common pivot point.
- **Data minimization**: Data you don't collect can't be leaked. When the code stores or transmits data, ask: *do we actually need this field?*

### 4. Dependency Trust

**Question**: What am I trusting by adding or updating this dependency, and is that trust justified?

**Key signals**:
- **Transitive trust**: Adding a dependency means trusting not just that package, but its entire dependency tree. A compromised transitive dependency five levels deep runs with your application's full privileges. `npm audit`, `pip audit`, and similar tools check the tree — not just direct dependencies.
- **Maintenance signals**: Check last commit date, number of maintainers, open issue response time. A package with one maintainer who hasn't committed in two years is a supply chain risk — the maintainer's account gets compromised, or the package gets transferred.
- **Permission scope**: Does the dependency need the access it requires? A color-formatting library that requests network access or filesystem access is suspicious.
- **Version pinning and lockfiles**: Unpinned versions (`^`, `~`, `*`) or missing lockfiles mean builds are not reproducible and a compromised version can enter automatically. Pin exact versions, use lockfiles (`package-lock.json`, `Cargo.lock`, `poetry.lock`), and review dependency updates before merging.
- **Auto-merge risk**: Automated dependency update tools (Dependabot, Renovate) reduce toil but must be reviewed, not auto-merged. A compromised package version is exactly what these tools would propose merging.

### 5. Secrets and Cryptography

**Question**: Is this code handling secrets correctly, and are cryptographic choices sound?

**Key signals**:
- **Secrets in source control**: API keys, database passwords, encryption keys, and tokens must never appear in committed code — not even temporarily, not in "private" repos. Git history preserves everything. If a secret was committed, it is compromised: rotate immediately. Removing it from history is necessary but not sufficient.
- **Secret storage**: Use environment variables or a secrets manager (Vault, AWS Secrets Manager, etc.). Hardcoded secrets in code, config files committed to the repo, or secrets in Docker images are all the same problem: the secret is accessible to anyone who can read the source.
- **Custom cryptography**: The question is not "does this algorithm look right?" but "why is this code implementing cryptography instead of using an established library?" Custom implementations of encryption, hashing, MAC computation, or token generation are almost always wrong. Use `libsodium`, framework-provided crypto, or well-maintained libraries.
- **Algorithm fitness**: Different operations require different algorithms. Password hashing needs slow algorithms (bcrypt, argon2). Data integrity needs HMAC. Encryption at rest needs AES-GCM or similar AEAD. Token generation needs a CSPRNG. Using the wrong algorithm (e.g., SHA256 for passwords, or `Math.random()` for tokens) is a category error, not just a bad choice.
- **Rotation and lifecycle**: Secrets and keys should have a rotation procedure that doesn't require downtime. If the code has no path for key rotation, it will never happen — and compromised keys will stay in production.

---

## Threat Modeling: When to Use STRIDE

STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) is a mental model, not a checklist. Use it when the code touches authentication, data access, or new interfaces and you need to systematically check whether you've missed a threat category.

**When to invoke it**: After you've worked through the relevant judgment dimensions and want to verify you haven't missed a category of threat. STRIDE is most valuable for new features, new APIs, and architectural changes — less useful for small patches to existing code where the threat model hasn't changed.

**How to use it in practice**: For each STRIDE category, ask one question:
- **Spoofing**: Can someone pretend to be someone else? (→ Dimension 2: Auth)
- **Tampering**: Can someone modify data they shouldn't? (→ Dimension 1: Trust Boundaries)
- **Repudiation**: Can someone deny they did something, and would we know? (→ audit logging)
- **Information Disclosure**: Can someone see data they shouldn't? (→ Dimension 3: Data Exposure)
- **Denial of Service**: Can someone exhaust resources? (→ rate limiting, unbounded queries, resource allocation)
- **Elevation of Privilege**: Can someone gain access beyond their role? (→ Dimension 2: Auth, specifically IDOR and privilege escalation)

If any answer is "I don't know" or "maybe," that's the signal to investigate further — not to assume it's fine.

---

## Failure Signals

Stop and reassess if:
- you are building queries with string concatenation of untrusted data
- you are storing or comparing passwords with anything other than a purpose-built slow hashing function
- you are trusting input without being able to point to where it's validated
- you are adding authorization checks in the UI but not at the API layer
- you are logging request bodies or headers without a sanitization filter
- you are adding a dependency without checking its maintenance status and dependency tree
- you are implementing custom cryptography instead of using established libraries
- you are hardcoding secrets or committing them to source control
- you cannot answer "who can reach this code path?" for a new endpoint

## Self-Check Before Exiting

- [ ] Have I identified the trust boundaries in this change and verified input is validated at each one?
- [ ] Can I state who can reach each new code path and confirm authorization is enforced at the right layer?
- [ ] Is sensitive data protected in transit, at rest, and in logs — with explicit filtering, not discipline?
- [ ] For new dependencies: have I checked maintenance status, known vulnerabilities, and the transitive dependency tree?
- [ ] Have I considered whether STRIDE threat modeling is warranted for this change?
- [ ] Am I exiting because security is genuinely addressed, or rationalizing?

**If any check fails, return to the relevant section before exiting.**
