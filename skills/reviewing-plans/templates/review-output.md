# Plan Review Output Template

Use this only for a complete formal plan review. Omit empty severity sections.

## Plan Assessment

[Restate the plan intent in 1-2 sentences.]

**Repo alignment:** [State whether the plan conflicts with repo instructions, architecture, conventions, or cited code evidence. Cite sources for conflicts.]

**Design fit:** [State whether the approach matches the stated intent and whether material assumptions or scope changes are unresolved.]

## Issues

### Critical

**1. [Short title]**
- **Layer:** Repo Alignment / Intent & Design / Consistency / Executability / Risk
- **Location:** `plan-file.md:line` plus repo evidence when relevant
- **What:** [Specific wrong assumption, decision, omission, or contradiction]
- **Why:** [Concrete failure mode if execution follows the plan unchanged]
- **Fix:** [Smallest correction that removes the failure]

### Important

**1. [Short title]**
- **Layer:** Repo Alignment / Intent & Design / Consistency / Executability / Risk
- **Location:** `plan-file.md:line`
- **What:** [Specific issue]
- **Why:** [Concrete rework, ambiguity, hidden coupling, or verification risk]
- **Fix:** [Targeted correction]

### Minor

**1. [Short title]**
- **Location:** `plan-file.md:line`
- **What / Fix:** [Clarity improvement that does not affect execution safety]

## Verdict

**Ready to implement?** `Approved` / `Approved with fixes` / `Blocked`

**Reasoning:** [One or two sentences naming the issue severity and layer that drives the verdict. Use `Blocked` for any Critical issue.]
