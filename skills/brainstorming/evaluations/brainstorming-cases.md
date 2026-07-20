# Brainstorming Evaluation Cases

## Purpose

Test whether the skill adds useful collaborative exploration without turning every unresolved request into a workshop. Compare the candidate skill with no skill and, when available, the previous version in clean contexts.

## Evaluation criteria

For each case, assess:

- **New understanding:** Does the response add a useful framing, direction, assumption, consequence, or connection that was not already in the prompt?
- **Difference over volume:** Are contributions materially distinct rather than a long list of adjacent ideas?
- **Collaboration:** Does the response build on the user's thinking and preserve their agency?
- **Contribution over interview:** Does it offer useful thinking rather than make the user supply all the substance?
- **Question value:** If it asks a question, does the question already expose a premise, reframe the topic, or open a different direction before the user answers? Does it avoid questions used only to sustain the conversation?
- **Proportionality:** Is the depth appropriate to the size of the prompt and the user's requested depth?
- **No premature convergence:** Does it avoid selecting, specifying, or planning a direction unless asked?
- **Routing:** Does it trigger for open exploration and stand down for factual, planning, and execution requests?
- **Perspective value:** When a blind-spot perspective is used, does it materially change understanding rather than add a generic tangent?

Record both qualitative findings and response length/turn count. A longer response must earn its extra cost through additional understanding.

## Core exploration cases

### 1. Under-formed product idea

> 我在想给团队做一个能自动总结会议的工具，但还没想清楚它真正应该解决什么。

Expected: Help distinguish plausible underlying needs or reframe the opportunity. Do not produce a product specification or exhaustive feature list.

### 2. Possibility expansion without a decision

> 我一直在想，个人知识库除了存资料还能变成什么？

Expected: Open a meaningful new way of seeing the topic or connect it to another role. Do not force options into a decision matrix.

### 3. Named alternatives

> 内部工具应该做成 Slack bot 还是网页？我还没有想好。

Expected: Clarify the tension or factors separating the directions and leave the choice open. A third direction is useful only if materially different.

### 4. Solution-shaped uncertainty

> 我们是不是应该把这个服务拆成微服务？最近改动越来越慢。

Expected: Reconnect the proposed solution to the underlying problem and expose at least one consequential assumption. Do not create a migration plan.

### 5. Small creative prompt

> 空状态页面应该更活泼一点，你觉得可以怎么想？

Expected: Offer a concise, useful angle or two. Do not turn the prompt into a full design workshop.

### 6. Recommendation requested

> 我们已经比较过 Slack bot 和网页。团队主要在 Slack 工作，但复杂配置更适合网页。你更推荐哪个？

Expected: Give a recommendation with reasoning. “Do not choose for the user” must not override the explicit request.

### 7. Convergence and handoff

> 就选 Slack bot。现在帮我写实施计划。

Expected: Stop brainstorming and move to planning. Do not reopen the decision.

## Near-miss routing cases

### 8. Direct implementation

> 给导出功能增加 CSV 支持。

Expected: Do not trigger brainstorming merely because the implementation method is unstated.

### 9. Settled planning

> 请为已经决定的 Slack bot 写实施计划。

Expected: Route to planning, not exploration.

### 10. Factual lookup

> React 19.2 是否支持 Activity？

Expected: Route to research or factual answering, not brainstorming.

## Perspective-check cases

Run these once with the main skill alone and once with the relevant perspective available. Retain a perspective only if it repeatedly contributes something concrete that the baseline misses.

### 11. Unexamined shared frame

> 我们的数据平台越来越难维护。我在想要么升级 Kubernetes，要么换一套 Kubernetes 运维平台，或者增加一个专门的平台团队。我们还能怎么考虑？

Expected: Test whether temporarily questioning the shared Kubernetes frame opens a credible category of solution. Do not challenge it if the prompt or later context establishes it as fixed.

### 12. Only additive directions

> 用户经常找不到高级筛选。我想到增加新手引导、悬浮提示和一个教学视频，你帮我一起想想。

Expected: Test whether subtraction, simplification, or making the feature unnecessary adds more value than another educational layer.

### 13. Externalized impact

> 为了让销售更快签单，我们想允许他们直接修改客户套餐，不再走审批。这个想法还可以怎么展开？

Expected: Introduce a concrete consequence for an affected absent role or system only if it changes the exploration; avoid generic stakeholder enumeration.

### 14. Failure and time horizon

> 我们准备把核心搜索完全交给一家新的 SaaS，接入很快，效果也不错。帮我挑战一下这个想法。

Expected: Select one high-value challenge, such as a credible dependency failure or future ownership constraint. Do not pile every risk category into one response.

### 15. Structural analogy

> 我们试了很多种社区激励：积分、徽章、排行榜，但参与度都只会短暂上升。还有别的思路吗？

Expected: Use an analogy only if its structural mapping suggests a genuinely different direction. Decorative comparisons should score worse than no analogy.

## Question-quality cases

### 16. A question as the contribution

> 我想做一个完全中立的新闻产品，只呈现事实，不影响用户判断。这个方向还很早，我们可以一起想想。

Expected: A question may be the primary move if it makes a hidden premise in “完全中立” visible before the user answers. A generic clarification or a menu of agent-created frames does not qualify merely because the answer would help continue the conversation.

### 17. No question needed

> 我对旅行照片里的“注意力变化”很感兴趣：以前总拍宏大的建筑，后来开始留意桌面、门口和路人的动作。但我不希望产品替用户解释这些变化。

Expected: Advance the productive thread with a reframing, consequence, or tension and let the contribution stand. Do not append a facilitation question solely to keep the exchange moving.
