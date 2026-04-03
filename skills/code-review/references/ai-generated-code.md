# Reviewing AI-Generated Code

AI-generated code requires the same review rigor as human-written code. The layered review model and all specialized lenses apply unchanged. This document covers reviewer-specific adjustments — what to look for differently when you know or suspect the code was AI-generated.

---

**Verify external references against source.** AI tools generate plausible-but-nonexistent APIs, incorrect method signatures, and outdated library interfaces. As a reviewer, you cannot trust that imports and API calls were validated by the author. Spot-check external API calls, especially for less common libraries — open the actual docs or source code rather than relying on your own familiarity.

**Increase skepticism of "looks right" code.** AI-generated code passes superficial reading more easily than human code — correct structure, reasonable names, sensible-looking flow — while containing subtle logic errors. Slow down at boundary conditions, off-by-one scenarios, and null/undefined handling. If the code looks too clean for its complexity, trace the edge cases manually.

**Search for duplicated functionality.** AI tools lack full codebase awareness and frequently generate functions that already exist elsewhere. Before accepting a new utility, helper, or abstraction, search the codebase for existing equivalents. This is a reviewer responsibility because the author (especially an AI-assisted one) may not have checked.

**Scrutinize test assertions.** AI-generated tests frequently mock the system under test, assert on implementation details, or pass regardless of behavior changes. Check: would this test fail if the behavior actually broke? Tests that exercise mocks instead of real behavior provide false coverage.
