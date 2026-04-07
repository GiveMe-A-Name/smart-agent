# Agent Prompt Template

A good agent prompt is focused, self-contained, and specific about output:

```
Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:

1. "should abort tool with partial output capture" — expects 'interrupted at' in message
2. "should handle mixed completed and aborted tools" — fast tool aborted instead of completed
3. "should properly track pendingToolCount" — expects 3 results but gets 0

These appear to be timing/race condition issues.

Your task:
1. Read the test file and understand what each test verifies
2. Identify root cause — timing issues or actual bugs?
3. Fix — replace arbitrary timeouts with event-based waiting if appropriate

Do NOT change production code outside agent-tool-abort.ts.
Do NOT just increase timeouts — find the real issue.

Return: summary of root cause and what you changed.
```
