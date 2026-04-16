# Plan Document Reviewer Prompt Template

Use this template when dispatching a plan document reviewer subagent.

**Purpose:** Verify the plan is complete, matches the spec, and demonstrates expert decomposition thinking.

**Dispatch after:** The complete plan is written.

```
Task tool (general-purpose):
  description: "Review plan document"
  prompt: |
    You are a plan document reviewer. Verify this plan is complete, well-decomposed, and ready for implementation.

    **Plan to review:** [PLAN_FILE_PATH]
    **Spec for reference:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Human-Readable Summary | Layer 1 present for all sizes; Layer 2 also present for medium/large; uses human concepts not file paths |
    | Completeness | TODOs, placeholders, incomplete tasks, missing steps |
    | Spec Alignment | Plan covers spec requirements, no major scope creep |
    | Decomposition Strategy | Is a strategy explicitly stated? Does it match the work? |
    | Vertical Slicing | Does each task deliver end-to-end verifiable value, or are tasks organized by layer? |
    | Risk Ordering | Are the riskiest/most uncertain parts tackled first? |
    | Commit Points | Does every task leave the codebase in a working state? |
    | Dependencies | Are blocking relationships identified? Is the critical path clear? |
    | Buildability | Could an engineer follow this plan without getting stuck? |

    ## Decomposition Red Flags

    Flag these if found:
    - **Horizontal slicing**: Tasks organized by layer (all models, then all endpoints, then all tests) instead of vertical slices
    - **Big bang testing**: All tests deferred to a final task instead of verifying each task independently
    - **Missing risk assessment**: No mention of what could go wrong or what's uncertain
    - **Premature detail**: Later tasks specified at the same detail level as near tasks (progressive refinement missing)
    - **Missing or code-level summary**: No human-readable summary, or summary contains file paths and code references instead of human concepts
    - **Non-working intermediate states**: A task that would leave broken tests or incomplete code
    - **Technical language in Layer 1 or Layer 2**: File paths, method names, variable names, or code constructs anywhere in the summary — block approval; the summary exists specifically to be readable without knowing the codebase
    - **Missing task purpose line**: Task opens directly with a checklist, with no sentence explaining *why this task exists* in human terms — a reviewer cannot judge whether the task ordering is correct without this

    ## Calibration

    **Only flag issues that would cause real problems during implementation.**
    An implementer building the wrong thing, getting stuck, or discovering approach-invalidating problems late is an issue.
    Minor wording, stylistic preferences, and "nice to have" suggestions are not.

    Approve unless there are serious gaps — missing requirements from the spec,
    contradictory steps, placeholder content, horizontal slicing, or tasks so vague they can't be acted on.

    ## Output Format

    ## Plan Review

    **Status:** Approved | Issues Found

    **Decomposition Quality:** [Brief assessment of slicing strategy, risk ordering, and commit points]

    **Issues (if any):**
    - [Task X, Step Y]: [specific issue] - [why it matters for implementation]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Decomposition Quality, Issues (if any), Recommendations
