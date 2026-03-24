# Example: Routing Language vs Boundary Language

This pair shows the same skill written two ways.
The bad version looks like it is defining a boundary. It is not — it is encoding a routing graph.
The good version defines what the skill owns and what it does not, without prescribing the next step.

---

## Bad Version

```markdown
---
name: fix-github-issue
description: Use when fixing a GitHub issue. Run after understand-codebase and before write-tests.
---

# Fix GitHub Issue

## Overview

This skill fixes GitHub issues. It is used after the codebase has been understood
and before tests are written.

## Workflow

1. Read the issue
2. Use understand-codebase skill to map relevant files
3. Implement the fix
4. Hand off to write-tests skill

## Relationship to Other Skills

- Depends on: understand-codebase
- Triggers next: write-tests
- Route back to understand-codebase if the fix scope changes
```

### What is wrong

**The boundary is fake.** Every section that looks like scope definition is actually
routing instruction. Remove the named skill references and nothing remains that
teaches what this skill owns or when it applies.

**The "Workflow" is a fixed sequence.** Steps 2 and 4 are handoffs, not capability.
An agent following this will route rather than judge — it will call `understand-codebase`
even when it already has the relevant context.

**The description encodes position in a pipeline**, not trigger conditions.
"Run after X and before Y" tells the agent where it sits in a graph,
not when the capability is needed.

**Self-check fails.** Delete all named skill references:
> *Use when fixing a GitHub issue. [...] Read the issue. Implement the fix.*

What remains is too thin to guide any real decision.

---

## Good Version

```markdown
---
name: fix-github-issue
description: Use when the task is to diagnose and fix a reported GitHub issue,
including understanding the failure, locating the relevant code, and verifying
the fix does not regress existing behavior.
---

# Fix GitHub Issue

## Capability Boundary

This skill owns: reading and understanding the issue, locating affected code,
implementing a targeted fix, and verifying the fix.

This skill does not own: setting up the repository, writing new feature tests
unrelated to the issue, or deciding whether the issue is worth fixing.
If repository context is missing, state what is missing rather than assuming it.

## Invariants

- Do not begin implementing before you can state what the failure is and where it occurs.
- Do not mark a fix complete without verifying it does not break existing tests.
- If the issue cannot be reproduced, report that explicitly rather than guessing at a fix.

## Decision Signals

When reading the issue, ask:
- Is the failure clearly described, or is it ambiguous? Ambiguity must be resolved
  before touching code.
- Is this a behavior regression, a missing feature, or a configuration problem?
  Each requires a different kind of fix.

When locating the code, ask:
- Is the failure caused by the code that looks most obviously related,
  or is it caused by something upstream?

## Failure Signals

Stop and re-evaluate if:
- the fix requires changing more than two unrelated areas of the codebase
- the issue cannot be reproduced with the information provided
- the fix passes tests but contradicts the stated behavior in the issue description
```

### What is better

**The boundary names what the skill owns and does not own** without pointing
to any other skill by name. An agent that has relevant context already can proceed;
one that is missing context knows exactly what to state before continuing.

**The invariants are red lines**, not steps. They prevent the agent from
taking the most common shortcuts: implementing without understanding,
and calling a fix done without verifying.

**The decision signals teach judgment at the two hardest moments**:
reading the issue (where ambiguity is easy to skip) and locating the code
(where the obvious candidate is often wrong).

**Self-check passes.** Delete all named skill references — there are none.
The file still teaches a complete, usable capability.

---

## The Core Distinction

Routing language answers: *what runs next?*
Boundary language answers: *what does this skill own, and what is missing when it cannot proceed?*

A skill that can only answer the first question is a pipeline node, not a capability.
