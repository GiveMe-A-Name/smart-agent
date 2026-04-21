# Plan Document Reviewer Prompt Template

Use this template when dispatching a plan document reviewer subagent during the writing-plans review loop.

**Purpose:** Independent review of the completed plan before handing off to the user.

**Dispatch after:** The complete plan is written and saved.

**When to dispatch:** Medium/large plans only — skip for tiny/small (overhead exceeds value).

```
Task tool (general-purpose):
  description: "Review plan document"
  prompt: |
    You are a plan document reviewer. Load the `reviewing-plans` skill and apply it to the plan below.

    **Plan to review:** [PLAN_FILE_PATH]
    **Spec for reference (if available):** [SPEC_FILE_PATH]

    The `reviewing-plans` skill defines the review layers, severity calibration, required context to read,
    and output format. Follow it exactly.

    Before reviewing, read CLAUDE.md, any architecture docs named in the plan,
    and any code files the plan references or assumes behavior from.
    A review without this context cannot catch repo alignment violations.

    Return the full structured review output as defined in the skill's output template.
```

**Reviewer returns:** Plan Assessment (Layer 0–1) · Issues (by severity) · Verdict (Approved / Approved with fixes / Blocked)

**On issues found:** Fix Critical and Important issues. Skip Minor. Stop the loop once no Critical/Important issues remain. If Critical/Important issues persist after 3 iterations, surface to the user — do not loop again.
