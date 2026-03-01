# Development Workflow Agent 设计问题讨论

---

## 完整上下文信息

### 原始设计文档

- **文件位置**: `/Users/qixuan/Projects/coordinator/plans/2026-02-27-development-workflow-agent-design.md`
- **内容**: 366 行的高层次设计文档
- **包含**:
  - 8 种 Agent 角色定义（主 Agent, Agent A-H）
  - 6 个阶段的流程描述（头脑风暴 → 评估 → 审核 → 用户确认 → 编码 → Code Review）
  - 上下文传递的基本原则
  - Skills 加载机制
  - 迭代控制和超时机制
  - MVP 版本规划

---

### 已完成的研究工作

#### 1. 上下文管理和通信机制研究（已完成）

**研究方向**:
- Multi-Agent 系统的上下文管理最佳实践（LangChain, AutoGen, CrewAI, MetaGPT）
- 智能上下文传递策略（增量同步、向量去重、上下文压缩）
- 受控的直接通信机制

**产出**:
- 完整的数据结构设计代码（三层 Context Map, InformationBlock）
- TransmissionStrategy 决策引擎设计
- 受控直接通信协议（AgentMessage, ApprovalManager）
- 错误处理和回退机制
- 阶段级共享机制（PhaseManager）

**状态**: 详细代码已生成，但尚未整合到原始设计文档

#### 2. 评审反馈（两轮）

**第一轮**: NEEDS REVISION
- 遗漏并行任务竞态条件、错误恢复、信息过期等场景
- 缺少具体数据结构、决策逻辑、协议细节

**第二轮**: 需要修改（正在等待完整代码整合）

---

### 本次发现的使用问题（核心问题）

## 问题 1：Coordinator 跳过 sub-agents 直接进入设计

**现象：**
- 在头脑风暴阶段结束后，Coordinator 直接开始进行设计
- 没有按照流程启动 sub-agents（Agent A/B）进行方案研究

**可能原因：**
- Coordinator 判断需求过于简单，认为没必要进行完整的设计流程
- 缺乏一个明确的"是否需要进入下一阶段"的判断机制

**建议：**
- 明确进入下一阶段的条件（如：需求复杂度、预估工作量、用户明确要求等）
- 即使需求简单，也应该有一个轻量级的评估流程，而不是完全跳过

---

## 问题 2：Researcher 和 Evaluator 没有明确的先后顺序

**现象：**
- 在研究阶段，Researcher 和 Evaluator 的协作流程不清晰
- Evaluator 直接评估了主 Agent 的规划，而不是 Researcher 的产出

**预期流程：**
```
Researcher → 初步方案 → Evaluator 评估 → 循环迭代
```

**当前问题：**
- Researcher 输出后，没有明确的"交付给 Evaluator"步骤
- Evaluator 评估的对象不明确

**建议：**
- 明确 Researcher 的输出格式（应该是简洁的可行性方案，不是详细设计）
- 添加"交付"动作：Researcher 完成后 → 主 Agent 转发给 Evaluator → Evaluator 评估
- 明确 Evaluator 的评估对象是 Researcher 的产出，而不是主 Agent 的规划

---

## 问题 3：Researcher 输出的方案过于详细

**现象：**
- Researcher 直接输出了包含设计细节的完整方案
- 这应该是 Planner（Agent C）的工作，而不是 Researcher 的职责

**问题分析：**
- Researcher（研究阶段）的职责应该是：调研方案、评估可行性、提供选择
- 过于详细的方案会：
  - 让 Evaluator 难以进行客观评估（被细节绑定）
  - 抢占了 Planner 的工作
  - 增加迭代成本（每次修改都涉及大量细节）

**建议：**
- 明确 Researcher 输出的格式：
  - 方案概述（1-2 句话）
  - 技术路线（可选 1-2 个）
  - 关键风险点
  - 推荐的决策方向
- 详细设计应该留给 Planner 阶段

---

## 问题 4：Evaluator 反馈没有返回给 Researcher 进行修订

**现象：**
- Evaluator 评估后给出"不通过"和修改意见
- 修改意见没有再次交付给 Researcher 进行修订
- 流程在 Evaluator 拒绝后就停止了

**预期流程：**
```
Evaluator 不通过 → 反馈给 Researcher → Researcher 修订 → 再次评估（循环）
```

**当前问题：**
- Evaluator 的反馈停留在主 Agent 层面
- 没有形成 Researcher → Evaluator → Researcher 的迭代闭环

**建议：**
- 建立明确的迭代机制：
  1. Evaluator 给出反馈和修改方向
  2. 主 Agent 将反馈转发给 Researcher
  3. Researcher 根据反馈修订方案
  4. 修订后的方案再次交付给 Evaluator
  5. 循环直到 Evaluator 通过或达到最大迭代次数

---

## 问题 5：第三阶段 Planner 和 Reviewer 职责混淆

**现象：**
- 在阶段 3（审核规划）中，我错误地让 Planner 去"审核"设计方案
- 这导致 Planner 做了 Reviewer 应该做的工作

**预期流程：**
```
阶段 2: 方案评估 → Evaluator 通过
         ↓
阶段 3: 规划制定
         ↓
    Planner 创建详细实施计划（基于设计方案）
         ↓
    Reviewer 审核 Planner 输出的实施计划
         ↓
阶段 4: 用户确认
```

**问题分析：**
- **Planner（Agent C）职责**：阅读代码理解上下文，根据解决方案制定详细的实施计划
- **Reviewer（Agent D）职责**：审核 Planner 输出的规划是否完整覆盖解决方案
- 两者是"制定者 ↔ 审核者"的关系

**根本原因：**
- 原始设计文档中的流程描述不够清晰
- 我（Coordinator）在执行时理解错误

**建议：**
- 在设计文档中明确区分 Planner 和 Reviewer 的职责
- 确保 Planner 输出详细实施计划，Reviewer 只负责审核
- Reviewer 的审核标准应该是：规划是否完整覆盖解决方案

---

## 问题 6：Evaluator 评审规则过于严格

**现象：**
- Evaluator 对方案的评估标准过于严格
- 任何问题都会导致"不通过"，即使是小问题

**问题分析：**
- 当前设计：Evaluator 只输出"接受"或"拒绝"两种结论
- 问题：没有根据风险级别区分对待

**建议：**
- 引入风险级别分类：
  - **高风险**：可能导致系统崩溃、数据丢失、安全问题 → 必须修复
  - **中风险**：功能不完整、性能严重下降 → 建议修复
  - **低风险**：代码风格、文档问题 → 可接受
- **通过规则**：如果没有中高风险问题，则算通过
- 低风险问题可以记录为"建议改进"，但不阻塞流程

**评审结论格式改进：**
```markdown
## 评估结果

### 风险级别分类
- 高风险 🔴: [问题列表]
- 中风险 🟡: [问题列表]
- 低风险 🟢: [问题列表]

### 最终结论
- 如果无中高风险 → **接受**
- 如果有中高风险 → **需要修改**

### 建议改进（不阻塞）
- [低风险问题列表]
```

---

## 总结：需要补充的设计内容

1. **阶段准入/准出规则**
   - 什么条件下可以进入下一阶段
   - 什么条件下可以跳过某个阶段

2. **Agent 产出格式标准化**
   - Researcher 应该输出什么格式（简洁可行性方案）
   - Planner 应该输出什么格式（详细设计）
   - 避免职责越界

3. **明确的交付机制**
   - Researcher → Evaluator 的交付动作
   - Evaluator → Researcher 的反馈动作
   - 确保每次交接都有明确的信息传递

4. **迭代闭环**
   - 评估不通过时的处理流程
   - 反馈的格式化（修改方向要具体、可操作）
   - 最大迭代次数的控制

5. **明确 Planner 和 Reviewer 职责**
   - Planner = 制定详细实施计划
   - Reviewer = 审核规划是否覆盖解决方案

6. **Evaluator 风险分级评审**
   - 引入高/中/低风险分类
   - 无中高风险则通过
   - 低风险作为建议改进，不阻塞流程

---

## 待完成的工作

### 短期（本次优化）

1. **整合技术细节到设计文档**
   - 将上下文管理和通信机制的详细设计整合到原始文档
   - 或创建技术附录文档

2. **补充阶段流程细节**
   - 明确 Researcher → Evaluator 的交付机制
   - 明确 Evaluator → Researcher 的反馈机制
   - 标准化 Agent 产出格式

3. **增加阶段准入/准出规则**
   - 明确什么条件下进入/跳过某个阶段

4. **修正 Planner/Reviewer 职责**
   - 明确 Planner = 制定详细计划
   - 明确 Reviewer = 审核计划

5. **Evaluator 风险分级评审**
   - 引入高/中/低风险分类
   - 无中高风险则通过

### 长期（后续扩展）

1. 实现上下文管理和通信机制的具体代码
2. 增加性能测试阶段的详细设计
3. 完善根因分析流程

---

## 已完成的工作（本次会话）

### Sub-agent 直接通信机制设计

**状态**：已完成并通过审核

**产出文档**：
- `plans/2026-02-28-subagent-direct-communication.md` - 设计方案
- `docs/plans/2026-02-28-subagent-direct-communication-implementation.md` - 实施计划

**核心设计**：
- Sub-agents 采用轮转对话模式直接沟通
- 主 Agent 转型为观察者+资源池角色
- 对话记录持久化用于追溯和审计
- 引入安全词 `[ESCALATE]` 机制
- 迭代上限 2-3 轮

**8 个实施 Task**：
1. 更新 Coordinator Agent
2. 更新 Researcher Agent
3. 更新 Evaluator Agent
4. 更新 Planner Agent
5. 更新 Reviewer Agent
6. 创建 Coder Agent
7. 创建对话日志系统
8. 更新主设计文档

---

## 相关文件

- `plans/2026-02-27-development-workflow-agent-design.md` - 原始设计文档
- `discuss.md` - 本讨论文件
