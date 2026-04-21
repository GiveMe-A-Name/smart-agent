# Evaluation Criteria

Five dimensions for evaluating a review comment before deciding Accept / Push back / Clarify. Apply relevant ones — not a fixed sequence.

## 1. Problem-Driven vs. Preference-Driven

Classify before anything else.

| Signal | Classification | Default |
|--------|---------------|---------|
| "Crashes in X scenario", "security hole", "returns wrong result for Y" | Problem-driven | Evaluate further |
| "I'd name this differently", "cleaner", "I prefer this style" | Preference-driven | Author's call — Push back unless style guide mandates |

## 2. Consequence Test

Ask: **if this is not fixed, what happens?**

- Bug / security issue / measurable degradation → should be addressed
- Personal preference / aesthetic difference → optional

## 3. Evidence First

Before any judgment:
1. Read the code being commented on
2. Grep for actual usage patterns referenced in the comment
3. Locate design intent source — commit messages, PR description, prior conversation

Judge against code evidence, not the comment's framing.

## 4. Scope Classification

| Scope | Threshold | Action |
|-------|-----------|--------|
| Implementation-level (local logic, naming, bug fix) | Standard | Decide after evidence check |
| Architecture-level (layer boundaries, module responsibilities, global patterns) | High — requires clear design intent conflict to push back | Escalate to human partner if intent is unclear; if no human partner is reachable, default to Clarify |

## 5. Value Assessment

Before accepting, verify both:

- **Benefit**: Does it fix a real problem? Improve correctness, clarity, or maintainability?
- **Cost + Risk**: Churn volume? Regression risk? Complexity added?

Accept only when benefit clearly outweighs cost + risk.
