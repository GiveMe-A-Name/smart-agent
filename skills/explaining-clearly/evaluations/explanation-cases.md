# Explaining Clearly Evaluation Cases

## Status and Evidence

Behavioral benefit is **unverified**. These are acceptance cases for a candidate skill, not a record of successful runs.

The capability gap comes from repeated conversation corrections: requests for concrete explanations after abstract answers; missing links between a change and why it works; useful reasoning compressed while peripheral detail remains; and follow-up corrections causing the whole topic to be explained again. The user also distinguishes understanding the current problem from accumulating knowledge through unrelated exercises.

## Comparison Method

Run each user prompt in its own clean context with and without the candidate skill. Keep model, tools, available source material, and other instructions the same. Give the executing agent only the user prompt, supplied material, and the skill path when applicable. Do not give it these criteria, a reference answer, previous outputs, or conclusions from another run.

Have a reviewer compare anonymized outputs against the criteria below. Record the actual prompts and context supplied, output length, unnecessary tool use, and concrete gains or losses in clarity. Do not score by keyword, heading count, or shortness alone. Use additional unseen cases and repeat comparisons before claiming a general improvement.

## Execution Cases

### 1. A mechanism behind an apparent contradiction

User prompt:

> 我们的服务一次只能处理一个请求，每个请求处理都需要 100 毫秒。原来请求大致隔 200 毫秒到达，现在经常同时来三个请求。大家说响应变慢了，但处理单个请求的代码没变。为什么会这样？

Reviewer criteria: Establish the difference between processing time and waiting time through the supplied situation. Make the sequence understandable without requiring the reader to already know queueing terminology. Do not invent a CPU, network, or database bottleneck or turn the answer into an optimization plan.

### 2. Explain why a change solves the problem

User prompt:

> 原来报表直接读取正在写入的订单表，不同查询可能读到不同时间的数据。现在先把某一时刻的数据复制到快照表，再让报表的所有查询都读这份快照，生成期间快照不再改。我知道改了什么，但为什么这样能避免报表里的数字互相对不上？

Reviewer criteria: Connect the common fixed dataset to agreement between queries, using the reader's reporting problem as the entry point. Explain the relevant consistency guarantee without asserting that snapshots prevent incorrect calculation logic or all data errors. Do not merely rename the solution as isolation or consistency.

### 3. Correct a particular inference

User prompt:

> 前面我们说过，缓存能复用已经算出的结果。我理解成只要用了缓存，请求就一定更快了。这一步理解对吗？

Reviewer criteria: Preserve the valid idea of reuse and address the leap to an unconditional guarantee. Give enough concrete explanation to resolve that leap. Do not restart a general introduction to caches, broadly validate the whole claim, or add a quiz.

### 4. Explain a distinction in an actual situation

User prompt:

> 我已经把错误账单发给客服，也知道三天后还没回就再联系。你说这件事可以算“已处理、等待中”，但钱还没退，怎么能叫已处理？

Reviewer criteria: Resolve the distinction between completing the available action and obtaining the final outcome using this situation. Keep the answer centered on the user's question. Avoid diagnosis, an unrelated analogy, a productivity system, or exercises for improving general understanding.

### 5. A crucial connection is missing from the source

User prompt:

> PR 描述只写了“给权限查询增加缓存，减少重复计算”，没有其他材料。你能解释这为什么不会导致权限变更后仍然读到旧结果吗？

Reviewer criteria: Recognize that the supplied description cannot establish the requested guarantee. Explain which missing information matters without asserting an invalidation or update mechanism exists. A plausible mechanism may be offered only as a clearly conditional illustration, not as the actual implementation.

## Discovery and Adjacent Tasks

Evaluate selection using only the skill's name and description alongside the other available skill descriptions. Then separately test execution where selection is appropriate.

| User request | Expected selection |
| --- | --- |
| “为什么改了这里，就能解决刚才的问题？” | Trigger: the relationship between change and result needs explanation. |
| “这一段我没看懂，尤其是从第二句话到第三句话。” | Trigger: a specific connection is missing for the reader. |
| “这句话只翻译成中文：Requests may wait before processing.” | Do not trigger: translation alone. |
| “把这份会议记录压缩成三条摘要。” | Do not trigger: summarization alone. |
| “只告诉我这个配置的默认值。” | Do not trigger: lookup alone. |
| “帮我修复这个请求超时的问题。” | Do not trigger merely because debugging involves understanding a mechanism. |
| “把这份故障说明改成面向客户的正式公告。” | Do not trigger merely because the document could use clearer prose. |

## Acceptance Question

Does the candidate make the important connection easier to understand while preserving accuracy and the user's task, at an acceptable cost in length and effort? If both versions already do this equally well, treat the benefit as unproven and reconsider the extra instruction.
