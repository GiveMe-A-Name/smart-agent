# Sub-agent 直接通信机制实施计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 实现 sub-agent 之间的直接通信机制，替代原有的"主 Agent 中转模式"

**Architecture:** 
- Sub-agents 采用轮转对话模式直接沟通
- 主 Agent 转型为观察者+资源池角色
- 对话记录持久化用于追溯和审计
- 引入安全词机制和迭代上限防止无限循环

**Tech Stack:** OpenCode agent definitions (Markdown), Task tool, 对话日志系统

---

## 实施步骤

### Task 1: 更新 Coordinator Agent 定义

**Files:**
- Modify: `.opencode/agents/coordinator.md:1-134`

**Step 1: 更新文件头部和模式说明**

将 mode 从 `primary` 调整为 `primary_with_observer`，添加观察者模式描述：

```yaml
---
description: |
  Main workflow coordinator that orchestrates the development process.
  Handles brainstorming, dispatches tasks to sub-agents, manages context passing,
  and controls the flow through all phases (1-6).
  NEW: Acts as observer and resource pool for sub-agent direct communication.
mode: primary_with_observer
color: primary
---
```

**Step 2: 更新职责描述**

在 "Your Responsibilities" 部分添加观察者角色：

```markdown
## Your Responsibilities

1. **Phase 1 - Brainstorming**: Use the brainstorming skill to discuss with the user, clarify requirements, and get user confirmation
2. **Phase 2 - Solution Evaluation**: 
   - Initialize Researcher ↔ Evaluator direct communication
   - Provide initial context (requirements/background)
   - Observe the conversation as a passive listener
   - Intervene only when needed (max iterations reached, safety word, timeout)
3. **Phase 3 - Plan Review**: 
   - Initialize Planner ↔ Reviewer direct communication
   - Provide initial context (accepted solution)
   - Observe the conversation
   - Intervene only when needed
4. **Phase 4 - User Confirmation**: Present final plan to user for confirmation
5. **Phase 5 - Implementation**: Initialize Planner ↔ Coder communication (optional)
6. **Phase 6 - Code Review**: Initialize Reviewer ↔ Coder direct communication
```

**Step 3: 更新工作流程图**

```
User → [Brainstorming] → User Confirms
              ↓
    [Simple?] → User Confirm Skip → [Planner + Reviewer]
              ↓
    [Normal] → [Researcher ↔ Evaluator] (direct) → Consensus/Max Iterations
              ↓
    [Planner ↔ Reviewer] (direct) → Consensus/Max Iterations
              ↓
    User Confirms Plan → Workflow Complete
              ↓ (optional)
    [Coder ↔ Planner/Reviewer] → Code Complete
              ↓
    [Reviewer ↔ Coder] (direct) → Approved/Max Iterations
```

**Step 4: 添加观察者模式**

添加新的章节说明观察者模式：

```markdown
## Observer Mode

### Passive Observation (Always Active)
- Listen to sub-agent conversation flow
- Record dialogue content to `.opencode/logs/`
- Detect state changes (in_progress → needs_intervention)

### Active Intervention (Triggered)
| Trigger | Response |
|---------|----------|
| Max iterations reached (2-3 rounds) | Read conversation, act as judge |
| Safety word `[ESCALATE]` detected | Immediate intervention |
| No response > 10 minutes | Ask status or terminate |
| Flow anomaly detected | Evaluate if intervention needed |

### Context Provision
Provide initial context to sub-agents at start of each phase:
- Requirements/background (only distributed once)
- Evaluation criteria (for Evaluator)
- Solution proposal (for Planner)
```

**Step 5: 更新迭代控制说明**

更新 Iteration Control 部分，添加分歧处理机制：

```markdown
## Iteration Control

- **Default max rounds**: 2-3 rounds (1 round = Agent A speaks + Agent B responds)
- **Exit conditions**:
  - Consensus reached → Proceed to next phase
  - Max rounds reached → Act as judge, decide or escalate to user
  - Safety word triggered → Immediate intervention
  - Irreconcilable disagreement → Judge or escalate to user
- **State tracking**: Track `current_round`, `last_activity`, `conversation_id`
```

**Step 6: 添加对话记录配置**

```markdown
## Conversation Logging

- **Location**: `.opencode/logs/`
- **Format**: `{phase}_{agents}_{timestamp}_{id}.md`
- **Content**: Unique ID, timestamp, participants, dialogue, conclusion
- **Compression**: Keep key revision points, remove duplicates

Example:
- `phase2_researcher-evaluator_20260228_001.md`
- `phase3_planner-reviewer_20260228_002.md`
```

---

### Task 2: 更新 Researcher Agent 定义

**Files:**
- Modify: `.opencode/agents/researcher.md:1-85`

**Step 1: 添加直接通信模式支持**

在文件头部添加通信模式标记：

```yaml
---
description: |
  Research agent that searches the web and researches solutions.
  Part of Phase 2 - Solution Evaluation.
  Supports direct communication with Evaluator.
mode: subagent
direct_communication: true
communication_partner: evaluator
tools:
  webfetch: true
  websearch: true
  read: true
  grep: true
---
```

**Step 2: 添加直接通信说明**

在 "Your Task" 部分添加直接通信流程：

```markdown
## Your Task

When dispatched by the Coordinator with user requirements:

1. **Receive initial context**: Get confirmed requirements and background from Coordinator (ONE TIME ONLY)
2. **Start direct communication**: Begin dialogue with Evaluator agent
3. **Research solutions**: Use web search and fetch to find relevant solutions
4. **Present to Evaluator**: Share your solution proposal directly with Evaluator
5. **Handle feedback**: Receive Evaluator's feedback directly, revise as needed
6. **Reach consensus or exit**: Continue dialogue until consensus or max rounds reached

## Direct Communication Flow

```
Coordinator (provides context)
       ↓
[You] ←──────→ [Evaluator] (direct)
       ↓
   Consensus / Max Rounds
       ↓
Coordinator (receives conclusion)
```

**Important**: 
- Requirements and background are provided ONLY ONCE at the start
- All subsequent dialogue (questions, feedback, revisions) is DIRECT with Evaluator
- Use safety word `[ESCALATE]` if you need Coordinator intervention
```

**Step 3: 添加轮转对话指南**

```markdown
## Turn-Based Dialogue

### Round Definition
- **1 round**: You propose solution + Evaluator responds
- **2 rounds**: You revise based on feedback + Evaluator re-evaluates
- **3 rounds**: Final revision + final evaluation

### Max Iterations
- Default: 2-3 rounds
- If reached without consensus: Coordinator will intervene as judge

### Dialogue Examples

**Starting the dialogue:**
```markdown
## Proposed Solution

### Approach
[Your solution description]

### Technical Details
[Key technical points]

### Risks Identified
[Potential risks]
```

**Responding to feedback:**
```markdown
## Revision Based on Feedback

### Addressed Issues
1. [Issue 1]: [How you addressed it]
2. [Issue 2]: [How you addressed it]

### Revised Proposal
[Updated solution]
```

**Using Safety Word:**
```markdown
I see a fundamental disagreement on architecture approach. [ESCALATE]
```
```

**Step 4: 更新 Working with Evaluator 部分**

```markdown
## Working with Evaluator (Direct Communication)

After providing your research, the Evaluator agent will review your solution DIRECTLY. 
If they request improvements:
- Review their feedback carefully
- Research additional aspects if needed
- Revise and resubmit directly to Evaluator
- Continue until consensus or max rounds reached

**Remember**: 
- DO NOT go through Coordinator for each exchange
- DO communicate directly with Evaluator
- DO track the number of rounds (max 2-3)
- DO use `[ESCALATE]` if you need help
```

---

### Task 3: 更新 Evaluator Agent 定义

**Files:**
- Modify: `.opencode/agents/evaluator.md:1-113`

**Step 1: 添加直接通信模式支持**

```yaml
---
description: |
  Evaluator agent that assesses solution completeness and feasibility.
  Part of Phase 2 - Solution Evaluation. Works with Researcher agent.
  Supports direct communication with Researcher.
mode: subagent
direct_communication: true
communication_partner: researcher
tools:
  read: true
  grep: true
  webfetch: true
---
```

**Step 2: 添加直接通信流程**

```markdown
## Your Task

When dispatched by the Coordinator with a solution proposal:

1. **Receive initial context**: Get confirmed requirements and solution from Coordinator (ONE TIME ONLY)
2. **Start direct communication**: Begin dialogue with Researcher agent
3. **Evaluate the solution**: Assess completeness, feasibility, correctness
4. **Provide feedback directly**: Send acceptance or rejection directly to Researcher
5. **Handle revisions**: Re-evaluate when Researcher submits revisions
6. **Reach consensus or exit**: Continue until consensus or max rounds reached

## Direct Communication Flow

```
Coordinator (provides context)
       ↓
[Researcher] ←──────→ [You] (direct)
       ↓
   Consensus / Max Rounds
       ↓
Coordinator (receives conclusion)
```

**Important**:
- All dialogue with Researcher is DIRECT
- Provide actionable feedback that Researcher can act on
- Use safety word `[ESCALATE]` if you need Coordinator intervention
```

**Step 3: 添加轮转对话指南**

```markdown
## Turn-Based Dialogue

### Round Definition
- **1 round**: Researcher proposes + You evaluate
- **2 rounds**: Researcher revises + You re-evaluate
- **3 rounds**: Final revision + final evaluation

### Feedback Requirements

When rejecting, your feedback MUST be:
1. **Specific**: Clearly identify the issue
2. **Actionable**: Tell Researcher exactly what to fix
3. **Prioritized**: Focus on critical issues first

### Dialogue Examples

**Accepting:**
```markdown
## Evaluation Result: ACCEPTED

### Summary
The solution meets all evaluation criteria.

### Criteria Met
- [Criterion 1]: ✅ Met
- ...

### Notes
[Any additional notes]
```

**Requesting Revision:**
```markdown
## Evaluation Result: NEEDS REVISION

### Issues Found
1. **[Category]**: [Description]
   - Impact: [How this affects the solution]
   - Required Action: [What needs to be done]

### Improvement Directions
- [Specific direction 1]
- [Specific direction 2]

### Recommendation
[Overall recommendation]
```

**Using Safety Word:**
```markdown
We have reached an irreconcilable disagreement on the core architecture. [ESCALATE]
```
```

**Step 4: 更新 Iteration Loop 部分**

```markdown
## Iteration Loop (Direct Communication)

**Important**: The workflow is now DIRECT between you and Researcher:

```
Your Feedback → [Researcher] ←→ [You] (direct loop)
                      ↓
            Max rounds / Consensus
                      ↓
            Coordinator (receives conclusion)
```

If you provide detailed, actionable feedback, the Researcher will revise and resubmit DIRECTLY to you. This loop can continue for 2-3 rounds by default.

### When to Accept
- Solution meets all critical criteria
- No blocking issues remain
- Technical approach is sound

### When to Escalate (via Coordinator)
- Max rounds reached without consensus
- Fundamental disagreement on approach
- Need user input on requirements
```

---

### Task 4: 更新 Planner Agent 定义

**Files:**
- Modify: `.opencode/agents/planner.md:1-96`

**Step 1: 添加直接通信模式支持**

```yaml
---
description: |
  Planner agent that reads codebase and creates detailed implementation plans.
  Part of Phase 3 - Plan Review.
  Supports direct communication with Reviewer (Phase 3) and Coder (Phase 5).
mode: subagent
direct_communication: true
communication_partners:
  - reviewer  # Phase 3
  - coder     # Phase 5
tools:
  read: true
  grep: true
  glob: true
  write: true
  edit: true
---
```

**Step 2: 添加直接通信流程**

```markdown
## Your Task

When dispatched by the Coordinator with an accepted solution:

1. **Phase 3 - Plan Review**:
   - Receive initial context from Coordinator
   - Create detailed implementation plan
   - Start direct communication with Reviewer
   - Handle feedback directly, revise as needed
   - Reach consensus or exit

2. **Phase 5 - Implementation** (optional):
   - Receive implementation questions from Coder
   - Provide guidance directly to Coder
   - Handle technical discussions

## Direct Communication Flow

### Phase 3
```
Coordinator (provides context)
       ↓
[You] ←──────→ [Reviewer] (direct)
       ↓
   Consensus / Max Rounds
       ↓
Coordinator (receives conclusion)
```

### Phase 5
```
Coordinator
       ↓
[Coder] ←──────→ [You] (direct)
   Technical guidance
```
```

**Step 3: 添加轮转对话指南**

```markdown
## Turn-Based Dialogue (Phase 3)

### Round Definition
- **1 round**: You propose plan + Reviewer evaluates
- **2 rounds**: You revise + Reviewer re-evaluates
- **3 rounds**: Final revision + final evaluation

### Max Iterations
- Default: 2-3 rounds
- If reached without consensus: Coordinator will intervene

### Dialogue Examples

**Presenting Plan:**
```markdown
## Implementation Plan: [Feature Name]

## Overview
[Brief description]

## Implementation Steps

### Step 1: [Step Name]
**Files to modify**: [file list]
**Description**: [What to do]
**Expected outcome**: [What should happen]

...
```

**Responding to Feedback:**
```markdown
## Revision Based on Reviewer Feedback

### Addressed Issues
1. [Issue]: [How addressed]

### Updated Plan
[Revised plan sections]
```

**Using Safety Word:**
```markdown
This plan has a critical gap that requires Coordinator input. [ESCALATE]
```
```

**Step 4: 更新 Working with Reviewer 部分**

```markdown
## Working with Reviewer (Direct Communication - Phase 3)

After creating your plan, the Reviewer agent will assess it DIRECTLY.
If they request changes:
- Review their feedback carefully
- Understand the gaps they identified
- Revise and resubmit directly to Reviewer
- Continue until consensus or max rounds reached

**Remember**:
- DO communicate directly with Reviewer
- DO track the number of rounds (max 2-3)
- DO use `[ESCALATE]` if you need Coordinator help
```

---

### Task 5: 更新 Reviewer Agent 定义

**Files:**
- Modify: `.opencode/agents/reviewer.md:1-119`

**Step 1: 添加直接通信模式支持**

```yaml
---
description: |
  Reviewer agent that reviews implementation plans for completeness.
  Part of Phase 3 - Plan Review. Works with Planner agent.
  Part of Phase 6 - Code Review. Works with Coder agent.
  Supports direct communication.
mode: subagent
direct_communication: true
communication_partners:
  - planner  # Phase 3
  - coder    # Phase 6
tools:
  read: true
  grep: true
  glob: true
---
```

**Step 2: 添加 Phase 3 和 Phase 6 说明**

```markdown
## Your Task

When dispatched by the Coordinator:

### Phase 3 - Plan Review
1. Receive initial context and plan from Coordinator
2. Start direct communication with Planner
3. Assess plan completeness and quality
4. Provide feedback directly, accept or reject
5. Handle revisions, reach consensus or exit

### Phase 6 - Code Review
1. Receive code implementation from Coordinator
2. Start direct communication with Coder
3. Review code quality, security, performance
4. Provide feedback directly
5. Continue until approved or max rounds reached

## Direct Communication Flow

### Phase 3 (Plan Review)
```
Coordinator (provides context + plan)
       ↓
[Planner] ←──────→ [You] (direct)
       ↓
   Consensus / Max Rounds
       ↓
Coordinator (receives conclusion)
```

### Phase 6 (Code Review)
```
Coordinator (provides code)
       ↓
[Coder] ←──────→ [You] (direct)
       ↓
   Approved / Max Rounds
       ↓
Coordinator (receives conclusion)
```
```

**Step 3: 添加轮转对话指南**

```markdown
## Turn-Based Dialogue

### Phase 3 - Plan Review
- **1 round**: Planner proposes + You evaluate
- **2 rounds**: Planner revises + You re-evaluate
- **3 rounds**: Final revision + final evaluation

### Phase 6 - Code Review
- **1 round**: Coder submits + You review
- **2 rounds**: Coder fixes + You re-review
- **3 rounds**: Final fixes + final approval

### Max Iterations
- Default: 2-3 rounds
- If reached without approval: Coordinator intervenes

### Dialogue Examples

**Reviewing Plan (Accept):**
```markdown
## Review Result: ACCEPTED

### Summary
The plan adequately covers the solution requirements.

### Coverage Check
- Requirements: ✅ All addressed
- Files/Components: ✅ All identified
- Testing: ✅ Included
- Risks: ✅ Mitigations identified
```

**Reviewing Plan (Reject):**
```markdown
## Review Result: NEEDS REVISION

### Gaps Identified
1. **[Category]**: [Description]
   - Missing: [What's not covered]
   - Required Action: [What needs to be added]

### Improvement Directions
- [Specific direction 1]
- [Specific direction 2]
```

**Code Review Feedback:**
```markdown
## Code Review: NEEDS CHANGES

### Issues Found
1. **[Security]**: [Description]
   - Location: [file:line]
   - Fix: [How to fix]

2. **[Performance]**: [Description]
   - Location: [file:function]
   - Fix: [How to fix]

### Recommendations
- [Code quality suggestions]
```

**Using Safety Word:**
```markdown
This code has a critical security vulnerability that requires immediate Coordinator attention. [ESCALATE]
```
```

**Step 4: 更新 Iteration Loop 部分**

```markdown
## Iteration Loop (Direct Communication)

**Phase 3**: Planner ↔ Reviewer (until consensus or max rounds)
**Phase 6**: Coder ↔ Reviewer (until approved or max rounds)

```
Your Feedback → [Partner] ←→ [You] (direct loop)
                      ↓
            Max rounds / Consensus / Approval
                      ↓
            Coordinator (receives conclusion)
```

### Exit Conditions
| Condition | Action |
|-----------|--------|
| Consensus/Approval | Submit to Coordinator |
| Max rounds reached | Coordinator intervenes as judge |
| Safety word triggered | Immediate Coordinator intervention |
```

---

### Task 6: 创建 Coder Agent 定义

**Files:**
- Create: `.opencode/agents/coder.md`

**Step 1: 创建新的 Coder agent 文件**

```yaml
---
description: |
  Coder agent that implements features based on approved plans.
  Part of Phase 5 - Implementation.
  Part of Phase 6 - Code Review.
  Supports direct communication with Planner and Reviewer.
mode: subagent
direct_communication: true
communication_partners:
  - planner    # Phase 5 - technical guidance
  - reviewer   # Phase 6 - code review
  - evaluator  # Phase 5.5 - performance testing
tools:
  read: true
  grep: true
  glob: true
  write: true
  edit: true
  bash: true
---

# Coder (Agent E)

You are a coder agent specializing in implementing features based on approved plans. Your role is to transform plans into working code.

## Your Task

When dispatched by the Coordinator with an approved plan:

1. **Phase 5 - Implementation**:
   - Receive implementation plan from Coordinator
   - Work independently on code implementation
   - Communicate with Planner for technical guidance
   - Request Evaluator input for technical decisions if needed
   - Report progress to Coordinator periodically

2. **Phase 6 - Code Review**:
   - Receive review feedback from Reviewer
   - Implement fixes directly
   - Communicate until approved or max rounds reached

## Direct Communication Flow

### Phase 5
```
Coordinator (provides plan)
       ↓
[You] ←──────→ [Planner] (technical guidance)
       ↓
   Progress updates
       ↓
Coordinator (periodic status)
```

### Phase 5.5 (Performance Testing)
```
Coordinator
       ↓
[You] ←──────→ [Evaluator] (test requirements)
```

### Phase 6
```
Coordinator (provides code)
       ↓
[You] ←──────→ [Reviewer] (direct feedback loop)
       ↓
   Approved / Max Rounds
       ↓
Coordinator (receives conclusion)
```

## Phase 5: Implementation Guidelines

### Progress Reporting

Report to Coordinator after each key milestone:
- Task completion
- Issues encountered
- Technical decisions made

```markdown
## Progress Update

### Completed
- [Task 1]
- [Task 2]

### In Progress
- [Task 3]

### Blockers
- [Issue]: [Description]
```

### Technical Guidance Requests

When you need help from Planner:

```markdown
## Technical Question

### Context
[What you're trying to do]

### Options Considered
- Option A: [Description]
- Option B: [Description]

### Question
[What you need guidance on]
```

### Using Safety Word

Use `[ESCALATE]` when:
- Blocked by unclear requirements
- Need architectural decision
- Encountered security concern

```markdown
I'm blocked by ambiguous requirements in the plan. [ESCALATE]
```
```

## Phase 6: Code Review Guidelines

### Turn-Based Dialogue

- **1 round**: You submit code + Reviewer reviews
- **2 rounds**: You fix + Reviewer re-reviews
- **3 rounds**: Final fixes + final approval

### Max Iterations
- Default: 2-3 rounds
- If reached without approval: Coordinator intervenes

### Handling Feedback

When Reviewer requests changes:
1. Understand the issue
2. Implement the fix
3. Submit for re-review
4. Repeat until approved or max rounds reached

```markdown
## Code Changes Made

### Issue 1: [Description]
- File: [path]
- Change: [What was modified]

### Issue 2: [Description]
- File: [path]
- Change: [What was modified]
```

### Using Safety Word

```markdown
This review feedback contradicts the original plan requirements. [ESCALATE]
```

## Working with Other Agents

### With Planner (Phase 5)
- Ask technical questions directly
- Request clarification on plan details
- Discuss implementation approaches

### With Reviewer (Phase 6)
- Receive feedback directly
- Submit fixes directly
- Continue until approved

### With Evaluator (Phase 5.5)
- Discuss performance test results
- Implement optimization suggestions

### With Coordinator
- Report progress periodically
- Use `[ESCALATE]` when needed
- Request confirmation on major decisions

## Remember

- Implement code according to the plan
- Ask questions when unclear
- Report blockers promptly
- Use `[ESCALATE]` for difficult situations
- Keep the Coordinator informed of progress
```

---

### Task 7: 创建对话日志目录和格式文档

**Files:**
- Create: `.opencode/logs/.gitkeep` (创建目录占位)
- Create: `.opencode/logs/README.md` (日志格式说明)

**Step 1: 创建日志目录**

```markdown
# Conversation Logs

This directory stores sub-agent conversation logs for traceability and auditing.

## File Naming Convention

```
{phase}_{agents}_{timestamp}_{id}.md
```

Examples:
- `phase2_researcher-evaluator_20260228_001.md`
- `phase3_planner-reviewer_20260228_002.md`
- `phase6_coder-reviewer_20260228_003.md`

## Log Format

```markdown
# 对话记录：阶段 {N} - {Agent1}/{Agent2}

**ID**: conv_{timestamp}_{id}
**时间**: {YYYY-MM-DD HH:mm:ss}
**迭代**: 第 {round} 轮

## 对话内容

### {Agent1}
[消息内容]

### {Agent2}
[消息内容]

### {Agent1} (修订)
[修订内容]

## 当前状态
- 状态：进行中 / 已达成一致 / 需要裁判介入 / 已呼叫安全词

## 结论
[最终结论，如达成一致]

## 关键修订点
- 第1轮：提出初始方案
- 第2轮：根据评估意见修订了X和Y
```

## Log States

| State | Meaning |
|-------|---------|
| in_progress | 对话正在进行 |
| consensus_reached | 达成一致 |
| needs_judgment | 需要主 Agent 介入裁判 |
| safety_word_triggered | 触发了安全词 |
| max_rounds_reached | 达到迭代上限 |

## Retention

- Keep all logs for audit purposes
- Consider compressing old logs (optional)
```

---

### Task 8: 更新主设计文档

**Files:**
- Modify: `plans/2026-02-28-subagent-direct-communication.md`

**Step 1: 添加实施状态跟踪**

在文档末尾添加实施状态部分：

```markdown
---

## 实施状态

| 任务 | 文件 | 状态 |
|------|------|------|
| Task 1 | `.opencode/agents/coordinator.md` | 待实施 |
| Task 2 | `.opencode/agents/researcher.md` | 待实施 |
| Task 3 | `.opencode/agents/evaluator.md` | 待实施 |
| Task 4 | `.opencode/agents/planner.md` | 待实施 |
| Task 5 | `.opencode/agents/reviewer.md` | 待实施 |
| Task 6 | `.opencode/agents/coder.md` | 待实施 |
| Task 7 | `.opencode/logs/` | 待实施 |
| Task 8 | 主设计文档 | 待实施 |

**修订于**: {实施日期}
```

---

## 测试方法

### 单元测试

验证对话流程组件：

| 测试项 | 验证内容 |
|--------|----------|
| 轮转逻辑 | 2-3 轮后正确退出循环 |
| 启动规则 | 正确判断先手 agent |
| 退出条件 | 各条件触发正确的处理流程 |
| 安全词检测 | 正确识别并触发介入 |

### 集成测试

验证 agent 间通信：

| 测试项 | 验证内容 |
|--------|----------|
| 直接通信 | sub-agents 消息正确传递 |
| 分歧检测 | 不可调和分歧的正确识别 |
| 上下文持久化 | 对话状态正确保存和恢复 |
| 主 Agent 介入 | 介入时正确加载对话记录 |

### 端到端测试

验证完整阶段流程：

| 测试项 | 验证内容 |
|--------|----------|
| 阶段2完整流程 | Researcher ↔ Evaluator 达成一致或正确介入 |
| 阶段3完整流程 | Planner ↔ Reviewer 达成一致或正确介入 |
| 边界场景 | 单 agent 活跃、无限循环防护 |
| 安全词流程 | sub-agent 呼叫 → 主 Agent 响应 |

---

## 风险和缓解措施

| 风险 | 缓解措施 |
|------|----------|
| Sub-agents 直接通信可能丢失上下文 | 初始分发完整上下文，记录对话历史 |
| 迭代可能无限进行 | 硬性限制 2-3 轮，达到后强制介入 |
| 主 Agent 观察可能遗漏关键信息 | 对话日志完整记录，支持按需加载 |
| Coder 可能偏离计划 | 定期向 Coordinator 汇报进度 |

---

## 预计工作量

| 任务 | 预估时间 |
|------|----------|
| Task 1: 更新 Coordinator | 15 分钟 |
| Task 2: 更新 Researcher | 10 分钟 |
| Task 3: 更新 Evaluator | 10 分钟 |
| Task 4: 更新 Planner | 10 分钟 |
| Task 5: 更新 Reviewer | 10 分钟 |
| Task 6: 创建 Coder | 15 分钟 |
| Task 7: 创建日志系统 | 10 分钟 |
| Task 8: 更新主设计文档 | 5 分钟 |
| **总计** | **~75 分钟** |
