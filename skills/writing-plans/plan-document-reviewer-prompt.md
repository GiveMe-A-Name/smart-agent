# Plan Document Reviewer Prompt Template

Use this template when dispatching a plan document reviewer subagent during the writing-plans review loop.

**Purpose:** Independent review of the completed plan before handing off to the user.

**Dispatch after:** The complete plan is written and saved.

**When to dispatch:** See Review Loop rules in `SKILL.md` — dispatch is conditional on plan size and Key Decisions count.

```
Task tool (general-purpose):
  description: "Review plan document"
  prompt: |
    You are a plan document reviewer. Load the `reviewing-plans` skill and apply it to the plan below.

    **Plan to review:** [PLAN_FILE_PATH]
    **Plan size:** [PLAN_SIZE]
    **Spec for reference (if available):** [SPEC_FILE_PATH]

    The `reviewing-plans` skill defines the review layers, severity calibration, required context to read,
    and output format. Follow it exactly.

    Before reviewing, read CLAUDE.md, any architecture docs named in the plan,
    and any code files the plan references or assumes behavior from.
    A review without this context cannot catch repo alignment violations.

    Return the full structured review output as defined in the skill's output template.
```

**Reviewer returns:** Plan Assessment (Layer 0–1) · Issues (by severity) · Verdict (Approved / Approved with fixes / Blocked)

**On issues found — blocking threshold differs by plan size:**
- **Large:** Fix Critical and Important issues. Skip Minor.
- **Medium:** Fix Critical issues only. Important issues must appear in the review output but do not block — note them for the human reviewer without triggering a fix loop.

Stop the loop once no blocking issues remain. If blocking issues persist after 3 iterations, surface to the user — do not loop again.
