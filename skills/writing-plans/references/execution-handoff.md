# Execution Handoff Reference

## Read-Only Constraints During Execution

These parts of the plan document have read-only constraints once execution begins:

1. **Decision Brief and any Design Review** — frozen separately at approval. Read them as the outcome and design contracts; never write to them silently. If execution requires a different outcome, scope, approved choice, boundary, contract, dependency direction, failure policy, migration, or rollout, stop and ask the human to approve a superseding section.

2. **Completed and in-progress tasks** — frozen as a historical record. Only not-yet-started tasks may be revised.

## Execution Log

Maintain an execution log at the bottom of the plan document, using the plan's language. Keep exact commands, paths, and code identifiers unchanged. Put the following short, labeled instructions directly in the generated plan, translated into its language. These bullets organize the instructions; log entries have no fixed field or table format.

```markdown
- **Language:** Write in the same language as this plan.
- **When to update:** After each task, before starting the next, append a dated entry identifying the task. Before handing off incomplete work, record progress and remaining work. Update this document, not only the chat.
- **What to write:** Explain in plain language what now happens under which conditions, or what the investigation established, and give actual verification results. Avoid abstract summaries such as "completed integration."
- **Final reconciliation:** Before final delivery, check the log against the tasks, promised behavior, and final checks. State unmet requirements and checks that failed or were not run.
- **Change boundaries:** Preserve approved decisions, designs, and started or completed task specifications. If the approved scope or design must change, propose the replacement and obtain confirmation.
```

Write for the human who approved the plan. Explain technical decisions through their practical effect, using code names or evidence links when needed. Record material plan revisions and their reasons; routine investigation and repair attempts need no separate account unless they explain the outcome or remaining risk.

## Final Reconciliation

Before final delivery, reconcile the saved log with the task list, promised behavior, and Final Verification. Fill missing entries from actual execution evidence; do not infer success from implemented code.

Account for each planned final check as passed, failed, or not run, with its command or an unambiguous reference to the Final Verification row. Checks with the same result may be grouped. If a narrower check replaced a planned check, identify both and explain the remaining coverage gap. A pre-existing failure is still a failed check; distinguish its cause from the current change without calling the full gate passed. Keep the final user-facing report consistent with this reconciliation.

## Plan Revision

If a plan revision trigger fires without changing an approved outcome or design: update not-yet-started tasks in the plan section and record the reason in the log. Do not rewrite completed or in-progress tasks.

If the revision changes approved content, leave the original Decision Brief or Design Review intact, add a clearly marked superseding approval section, and wait for approval before continuing.

## Execution Log Example

Illustrative entries for a Chinese plan:

```text
[2026-04-07] 任务 2：通知发送失败后最多重试两次，最终失败也不会把已经完成的任务改成失败。
`pnpm test tests/webhook.test.ts` 通过，确认总发送次数最多为三次，且通知失败不改变任务结果。

[2026-04-07] 最终验收：已交付通知开关、发送和有限重试，使用说明已更新。
最终验证表中的构建、通知测试和文档检查通过。
全仓 `pnpm lint` 失败，报错位于本次未修改的历史文件；变更文件检查通过，但不能替代全仓检查。
这一验收项仍未通过。
```
