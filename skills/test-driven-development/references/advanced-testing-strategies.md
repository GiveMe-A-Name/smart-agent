# Advanced Testing Strategies

**Load this reference when**: the core TDD cycle (red-green-refactor) is in place and the situation calls for techniques beyond example-based unit tests.

---

## Property-Based Testing

Instead of specifying individual input-output pairs, specify properties that should hold for ALL valid inputs. The test framework generates hundreds or thousands of random inputs automatically.

**Reach for it when:**
- The input space is large or combinatorial (parsers, serializers, data transformations)
- You find yourself writing many similar test cases that differ only in input values
- The code must satisfy mathematical properties (idempotency, commutativity, roundtrip encoding/decoding)
- Edge cases are hard to enumerate manually

**Key signals:**
- A property like "serialize then deserialize returns the original value" catches bugs that no finite set of examples would find
- "For any valid input, the output satisfies [invariant]" is more powerful than testing 5 specific inputs
- When a property-based test finds a failure, it shrinks the input to the minimal failing case — this IS the reproduction
- Property-based tests complement, not replace, example-based tests. Use examples for documentation and specific edge cases; use properties for broad correctness guarantees

---

## Contract Testing

When your code communicates across a boundary (API, message queue, shared library), both sides have expectations about the data format and behavior. Contract tests verify that these expectations remain aligned.

**Reach for it when:**
- Your code consumes an API owned by another team or service
- Your code provides an API consumed by others
- Integration tests are slow, flaky, or require complex environment setup
- You've been bitten by "it works in my test but breaks when the real service responds differently"

**Key signals:**
- Consumer-driven contracts: the consumer defines what it expects; the provider verifies it can meet those expectations. This catches breaking changes before deployment.
- Contract tests are NOT integration tests. They run fast, use no network, and verify the contract (data shape, required fields, error codes) independently of implementation.
- When adding a new field to an API response, contract tests tell you which consumers will break — before you deploy.

---

## Mutation Testing

Mutation testing answers the question: "If a bug existed in my code, would my tests actually catch it?" It works by making small changes (mutations) to the production code and checking if any test fails.

**Reach for it when:**
- You want to assess the quality of your test suite, not just coverage
- Code coverage is high but you suspect the tests are weakly asserted
- You are refactoring and want confidence that the test suite is a genuine safety net

**Key signals:**
- A surviving mutant (a code change that no test catches) reveals a gap in your test suite — either a missing test or a weak assertion
- 100% line coverage with 60% mutation score means 40% of potential bugs would go undetected
- Mutation testing is expensive to run. Use it periodically on critical code paths, not on every commit
- Focus on mutants that survive in business logic, not in boilerplate or glue code

---

## Testing Beyond Unit Tests

TDD starts at the unit level, but complete test confidence requires thinking about the full testing pyramid.

**Integration tests** verify that real components work together. Use them for: database interactions, external API calls (through a real test instance, not mocks), message queue behavior, file system operations.

**End-to-end tests** verify user-facing scenarios through the actual system. Use sparingly — they are slow and brittle. Each e2e test should map to a critical user journey that, if broken, would be immediately noticed.

**Testing in production** is not testing in the traditional sense — it's observability plus controlled rollout:
- Feature flags allow testing new behavior with a subset of users
- Canary deployments compare new code against old code with real traffic
- A/B tests verify that a change produces the expected outcome at scale
- These do not replace pre-deployment testing — they catch the things that pre-deployment testing cannot (scale effects, real user behavior, production data patterns)
