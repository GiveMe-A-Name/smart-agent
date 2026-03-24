# Compress to the Root Question

This contrast pair illustrates a specific failure mode that is different from
`resolve-vs-escalate.md`: when multiple unknowns share the same root, asking
each one separately scatters the user's attention instead of resolving the
situation.

The failure mode is listing downstream unknowns as separate questions when one
focused root question would unlock all of them.

---

## Scenario

**Request:** "那个慢查询太拖性能了，帮我优化一下"

A repository exploration reveals no business code — only skill definitions. No
query implementation exists to inspect.

---

## Bad Case

**Agent response:**

> "我已经探索了代码库，没有找到相关代码。需要澄清以下几点：
> 1. 哪个查询？请提供具体的查询语句
> 2. 什么场景？是数据库查询、API请求还是内部服务调用？
> 3. 有何性能数据？具体的响应时间或慢的原因"

**What went wrong:**

The three questions are not three separate unknowns. They are all downstream of
one root unknown: *the agent does not have the code*.

"哪个查询" asks which query. "什么场景" asks what kind of query. "有何性能数据"
asks how slow it is. All three are unanswerable without the code itself — and
once the code is provided, the agent can likely answer the second and third
questions itself.

Asking three questions when the situation calls for one gives the user a form to
fill in instead of a clear next step.

---

## Corrected Case

**Step 1 — Classify the unknowns:**

| Unknown | Type | Can self-resolve? |
|---|---|---|
| 哪段代码包含慢查询 | repository fact | No — no business code found in repo |
| 查询类型（DB / API / 内部调用） | repository fact | Yes, once code is available |
| 当前性能表现 | repository fact | Likely, once code and context are available |

**Step 2 — Identify the root unknown:**

All three unknowns share the same root: the agent does not have access to the
relevant code. Resolving the root unlocks the rest.

**Step 3 — Ask the one question that unlocks the rest:**

> "这个代码库中没有找到业务代码。请提供你要优化的查询代码，或者它所在的文件路径。"

Once the code is provided, the agent can determine query type, likely identify
the performance issue, and proceed without further questions.

**Why this is correct:**

One sentence answers the root gap. The downstream questions (type, performance
data) become self-resolvable once the code exists. Sending a list of three
questions implies three separate decisions the user must make — but the user
only needs to take one action.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Unknowns analyzed for shared root | No | Yes |
| Questions sent to user | 3, all downstream of same root | 1, directly at the root |
| User's required action | Fill in three answers | Provide the code |
| Remaining unknowns after user responds | Unknown | Self-resolvable from the code |

---

## When This Applies

When repo exploration turns up nothing, the instinct is to list everything
unknown. Resist this. Ask: what single piece of information, if provided, would
make the other unknowns self-resolvable? That is the root question. Everything
else is downstream noise.

The same principle applies when a request is extremely vague — "帮我规划一下这个功能怎么开发",
"fix the performance issue", "make it more maintainable". Multiple unknowns may
be visible, but there is usually one question whose answer makes the others
irrelevant or self-answerable: *which feature*, *which file*, *what behavior is
expected*. Find that question and ask only it.
