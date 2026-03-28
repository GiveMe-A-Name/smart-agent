# Thinking Mode Considerations

## Context

Some AI models support a "thinking mode" where the model performs internal reasoning in a `<thinking>` block before generating the visible response. This affects how skills should guide the agent's decision-making process.

## Skill Assessment in Thinking Mode

When the agent operates in thinking mode, skill invocation decisions can follow a two-phase pattern:

### Phase 1: Assessment in Thinking Block

The agent performs a structured assessment internally:

1. **Recognize the situation** — what type of task is this?
2. **Check skill applicability** — does a skill exist for this?
3. **Evaluate trigger conditions** — do the observable conditions match?
4. **Apply cost asymmetry** — what's the cost of false positive vs false negative?

This assessment happens in the thinking block and is not visible to the user.

### Phase 2: Conditional Output in Response

Based on the assessment, the agent outputs one of two paths:

- **If skill should be invoked**: Output the skill invocation tool call directly, with minimal or no explanatory text
- **If skill should not be invoked**: Proceed with the work directly, optionally noting why the skill was not needed (if relevant to the user)

## Benefits of This Pattern

1. **Reduces noise** — the user doesn't see lengthy explanations about whether to use a skill
2. **Preserves reasoning** — the thinking block captures the decision process for debugging
3. **Faster execution** — less back-and-forth about meta-decisions
4. **Clearer output** — the response focuses on the actual work, not the process

## Skill Design Implications

When designing skills for thinking-mode agents:

1. **Trigger conditions should be evaluable** — the agent needs clear criteria to assess in the thinking phase
2. **Cost asymmetry should be explicit** — helps the agent make the right default choice during assessment
3. **Observable conditions are critical** — the agent needs facts it can verify, not impressions
4. **Self-Check sections remain important** — they guide the exit decision, which also benefits from structured reasoning

## Example Flow

User request: "Add authentication to the API"

**Thinking block** (internal):
```
This is a new feature implementation. Let me check if I should use a planning skill.
- Is this non-trivial? Yes, authentication affects multiple files and has security implications.
- Do I have a complete plan? No.
- Cost of unnecessary invocation: brief planning pass.
- Cost of missing: building without considering security implications, missing edge cases.
- Decision: invoke the planning skill.
```

**Response** (visible):
```
[Tool call to planning skill]
```

## Non-Thinking Mode

For agents without thinking mode, the assessment must be more implicit or happen through explicit reasoning in the response. The skill's trigger conditions and cost asymmetry statements become even more critical, as they're the primary guide for the decision.

## Writing Skills for Both Modes

To support both thinking and non-thinking modes:

1. **Make trigger conditions explicit and observable** — works for both modes
2. **State cost asymmetry clearly** — helps both modes make the right default choice
3. **Keep invariants clear** — guides behavior regardless of reasoning style
4. **Design Self-Checks as structured questions** — can be answered in thinking block or explicitly

The core skill design principles (observable conditions, cost asymmetry, state-based triggers) work well for both modes. Thinking mode simply allows the assessment to happen more efficiently.
