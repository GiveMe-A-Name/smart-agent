---
name: agent-fix
description: Applies fixes to OpenCode agent definitions based on improvement plans. Use when: (1) Observer detects issues in agent behavior, (2) Agent definitions need updating, (3) Fix plans are ready to be applied. NOT for: skill fixes, user code changes.
---

# Agent Fix

Apply fixes to agent definition files in `.opencode/agents/`.

## Workflow

1. **Read target**: Load the agent .md file needing fixes
2. **Create snapshot**: Copy to `.opencode/snapshots/snapshot_{name}_{timestamp}.bak`
3. **Apply changes**: Use edit/write to apply the fix
4. **Verify**: Check syntax is valid markdown
5. **Report**: Summarize changes made

## Rules

**Allowed to modify:**
- `.opencode/agents/*.md`

**NEVER modify:**
- User project code
- External dependencies (node_modules, etc)
- Files outside `.opencode/`

## Rollback

If verification fails:
1. Find latest valid snapshot
2. Restore to original location
3. Report failure

## Output

Report: target file, changes applied, snapshot ID, status (SUCCESS/FAILED)
