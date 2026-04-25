---
name: using-skill-first
description: "Before any real work begins, check if an available skill encodes a better method. TRIGGER: session start before first action, or on task-type shift — approach commits happen early, and a skill invoked after approach is set cannot change it."
---

# Using Skills First

Capability ≠ best method. Skills encode methodology built from human failures at scale — your default approach is trained intuition, not validated practice. When they conflict, the skill is more likely right.

Skill misses happen when an agent starts a task it can do, not knowing a better systematic approach exists in available-skills.

## When to Check

Scan available-skills before any task involving:

- **Understanding** — reading unfamiliar code, forming assumptions about a system
- **Diagnosing** — something broken or behaving unexpectedly
- **Implementing** — writing behavior-changing code, making design decisions
- **Evaluating** — reviewing code, receiving feedback, judging a plan
- **Exploring** — no committed solution, multiple approaches open
- **Communicating** — artifacts others will read, decide from, or act on

*Boundary case*: a mechanical step with no analysis or judgment (e.g., renaming a variable) does not require a scan.

## Boundary

Owns: recognizing when a scan is warranted and invoking the best match.  
Does not own: executing the invoked skill's methodology.

## Invariants

- Scan before starting, not after an approach is set [a skill that changes approach must precede the commitment]
- "I already know how to do this" is a rationalization signal, not a skip reason [capability ≠ best method]
- The stronger the confidence in your approach, the more important the check — confidence is the most common mask for an unexamined assumption [you cannot evaluate what you haven't noticed]
- The check applies per action, not per session — session start does not discharge the obligation for subsequent steps
- Skip a candidate only with a specific, action-concrete reason — "not relevant" does not qualify [vague reasons can rationalize away any check; specificity makes the skip defensible]

## Completion Criteria

- [ ] Before each task in the six categories: was available-skills scanned?
- [ ] Any skipped candidate: was a specific, action-concrete reason stated?

## Verification

Dialogue-outcome type: observable from whether skill checks preceded task starts. Not self-verifiable from output alone.

## Anti-Rationalization Check

Did I skip a check because the task felt familiar or small? Familiarity and size are the most common skip rationales — and the two weakest.

## Failure Mode

Proceeding without checking. Once an approach is committed, an invoked skill cannot redirect it — it arrives after the decision, not before.
