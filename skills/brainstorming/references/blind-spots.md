# Brainstorming Blind Spots

Six blind spots humans systematically miss during brainstorming. Select 2–3 entries where the user's current framing shows the strongest matching detect signals. Inject each as a dedicated standalone question at the moment the signals appear. Each injected question counts as one of the one-question-per-message turns.

**Selection rule**: Prefer blind spots the user is least likely to reach on their own. Skip any whose detect signals are absent or that the user has already addressed unprompted.

---

## 1. Failure Blindness

**Detect when**:
- The conversation is focused on how to make the solution work, with no mention of how it could fail
- The team shows visible enthusiasm for a direction without anyone playing devil's advocate
- The solution has high failure cost (production systems, user-facing features, irreversible decisions)

**Inject this question**:
> Assume this solution fails six months after launch. What is the most likely cause?

**Skip when**: The user has already raised failure scenarios or risk cases unprompted.

---

## 2. Frame Lock

**Detect when**:
- The user treats a technology, architecture, or constraint as fixed without explaining why (e.g., "we have to use X", "we're building on top of Y")
- All options proposed so far are modifications of the same existing system rather than alternatives to it
- A reference design, competitor, or prior implementation was introduced early and options cluster around it

**Inject this question**:
> If we threw [X] away entirely and started from scratch, what different choices would we make?

Replace [X] with the specific system, constraint, or reference that appears to be anchoring the discussion.

**Skip when**: The user has already questioned a major constraint or proposed a fundamentally different direction.

---

## 3. Missing Perspectives

**Detect when**:
- The discussion only considers the primary user or the team's own goals
- The solution has clear downstream impact on other teams, systems, or user groups not present in the discussion
- There are identifiable parties who become worse off if the solution succeeds

**Inject this question**:
> Who is not in this discussion but will be most affected by this decision? What would their perspective be?

**Skip when**: The user has already named and considered stakeholders outside the immediate team or primary user.

---

## 4. Inversion Blindness

**Detect when**:
- All options on the table are positive implementations — no one has questioned whether to build this at all
- The user has stated "we will build X" without explaining why X needs to exist
- The problem might be simpler to solve by removing something than by adding something

**Inject this question**:
> Does this problem actually need to be solved? If we do nothing, what happens in six months?

If "do nothing" has already been ruled out, use this instead:
> If the goal were to make this as simple as possible, what would we cut?

**Skip when**: The user has already considered not building this, or has a clear stated reason why action is required.

---

## 5. Time Compression

**Detect when**:
- The solution looks obviously good in the short term but introduces dependencies, maintenance burden, or technical debt
- The decision involves third-party dependencies or external systems that could change
- The people making the decision now will not be the ones paying the long-term cost

**Inject this question**:
> What new problems will this decision create three years from now? Who will pay that cost?

**Skip when**: The user has already discussed long-term tradeoffs or explicitly acknowledged the maintenance cost.

---

## 6. Cross-Domain Analogy

**Detect when**:
- All options feel incremental — no genuinely different directions are on the table
- The problem belongs to a known category (coordination, queuing, caching, trust, incentive design, rate limiting, etc.) that other domains have solved
- The team's background is homogeneous, limiting exposure to outside approaches

**Inject this question**:
> How has this problem been solved in other domains (logistics, finance, gaming, healthcare...)? What can their approach teach us?

**Skip when**: The user has already drawn on analogies from outside their domain.
