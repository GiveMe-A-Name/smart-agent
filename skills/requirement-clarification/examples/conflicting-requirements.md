# Conflicting Requirements

This contrast pair illustrates a specific failure mode: when stated requirements
explicitly contradict each other, the clarification question is about priority or
tradeoff — not about what each requirement means individually.

The failure mode is treating conflicting requirements as separate clarification
items, which lets the conflict go unnamed and leaves the user without a clear
decision to make.

---

## Scenario

**Request:** "这个模块太乱了，给我重构一下，要保持简单，另外把所有边界情况都处理到"

---

## Bad Case

**Agent response:**

> "需要澄清几点：
> 1. 什么叫'保持简单'？是减少层级、缩短代码量，还是更清晰的命名？
> 2. 你说的'所有边界情况'具体是哪些？null 输入？空列表？并发？
> 3. 哪个模块需要重构？"

**What went wrong:**

The agent treated "保持简单" and "处理所有边界情况" as two separate unknowns
requiring independent clarification. They are not independent — they are in direct
tension. Comprehensive edge case handling adds branches, guards, and defensive
code; simplicity means fewer of those.

Asking what "simple" means and what edge cases exist does not surface this
tension. The user may answer both questions and still leave the agent with two
contradictory mandates and no way to resolve them.

The conflict is the thing to clarify — not the contents of each requirement.

---

## Corrected Case

**Step 1 — Classify the unknowns:**

| Unknown | Type | Can self-resolve? |
|---|---|---|
| Which module | repository fact | Possibly — check current directory context |
| What "simple" means | user intent | No |
| Which edge cases matter | user intent | Partially — common patterns are known |
| Priority when the two conflict | user intent | No |

**Step 2 — Identify the conflict:**

"保持简单" and "处理所有边界情况" trade off against each other. Every additional
edge case adds code paths. The more complete the coverage, the less simple the
result. These are not two requirements that can both be fully satisfied — one
constrains the other.

**Step 3 — Name the conflict and ask for the decision:**

> "这两个要求会互相制约：边界情况覆盖越全，代码越复杂；保持简单则意味着只处理最重要的情况。
> 你希望以哪个为主？还是有一个可以接受的折中（比如处理常见边界，忽略极端情况）？"

If the module is not identifiable from context, prepend: "另外，你指的是哪个模块？"
But the tradeoff question is the core — it is the one decision only the user can make.

**Why this is correct:**

The agent named the structural conflict before asking about either requirement's
details. The user now faces a real decision with real consequences, not a form
asking them to define terms. Once the priority is set, the scope of edge case
handling follows directly — no further clarification needed.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Conflict between requirements identified | No | Yes |
| User faces a real tradeoff decision | No | Yes |
| Questions sent to user | 3, about individual requirements | 1, about priority |
| Can agent proceed after user responds | Unclear — conflict still unresolved | Yes |

---

## When This Applies

When two stated requirements pull in opposite directions — "fast and thorough",
"flexible and simple", "backward-compatible and clean" — the clarification question
is not about what each requirement means. It is about which one governs when they
conflict. Ask that question directly.
