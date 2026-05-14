# Brainstorming Blind Spots

Six cognitive blind spots that systematically go unexplored in brainstorming conversations. Each entry describes why the blind spot matters, when to detect it, what question to inject, and when to skip.

**Selection rule**: Prefer blind spots the user is least likely to reach on their own. Inject only when detect signals match — do not work through the list. Each injected question counts as one of the one-question-per-message turns.

---

## 1. Failure Blindness

**Why it matters**: When a direction looks promising, attention fixates on making it work — failure modes become invisible because no one is looking for them. Forcing a concrete failure scenario reverses the search direction: instead of asking "how does this succeed?", the user must trace a specific path to failure. This surfaces hidden assumptions about reliability, edge cases, and dependencies that "make it work" thinking skips.

**Detect when**:
- The conversation is focused on how to make the solution work, with no mention of how it could fail
- The user shows commitment to a direction without naming any risk or failure case
- The solution has high failure cost (production systems, user-facing features, irreversible decisions)

**Inject this question**:
> Assume this solution fails six months after launch. What is the most likely cause?

**Skip when**: The user has already raised failure scenarios or risk cases unprompted.

---

## 2. Frame Lock

**Why it matters**: Once a system, tool, or reference design is treated as fixed, every option becomes a modification of that anchor. Whole categories of alternatives — replacing it, working around it, not using it at all — become invisible without ever being rejected. Asking what happens if the anchor is removed exposes which constraints are real and which are inherited from the frame, expanding the option space without forcing the user to abandon the anchor.

**Detect when**:
- The user treats a technology, architecture, or constraint as fixed without explaining why (e.g., "we have to use X", "we're building on top of Y")
- All options proposed so far are modifications of the same existing system rather than alternatives to it
- A reference design, prior implementation, or external example was introduced early and options cluster around it

**Inject this question**:
> If we threw [X] away entirely and started from scratch, what different choices would we make?

Replace [X] with the specific system, constraint, or reference that appears to be anchoring the discussion.

**Skip when**: The user has already questioned a major constraint or proposed a fundamentally different direction.

---

## 3. Missing Perspectives

**Why it matters**: A solution evaluated only from the user's vantage point optimizes for that vantage point — and silently externalizes cost onto parties who are not in the conversation. Naming an absent stakeholder forces the user to model a perspective that is not their own, which often reveals constraints, objections, or impacts that the original framing had no room for. The goal is not to give those parties a vote; it is to make their existence visible in the design space.

**Detect when**:
- The discussion only considers the primary user or the user's own goals
- The solution has clear downstream impact on other systems, users, or roles not present in the discussion
- There are identifiable parties who become worse off if the solution succeeds

**Inject this question**:
> Who is not in this discussion but will be most affected by this decision? What would their perspective be?

**Skip when**: The user has already named and considered stakeholders outside the immediate scope.

---

## 4. Inversion Blindness

**Why it matters**: Brainstorming defaults to additive thinking — what to build, what to add, what to change. The option of doing less, or doing nothing, is rarely placed on the table even when it is the strongest option. Inverting the framing exposes whether the problem actually needs to be solved at all, and whether the cheapest path is removal rather than construction. Even when the inversion is rejected, the rejection now carries an explicit reason instead of an unexamined assumption.

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

**Why it matters**: Decisions made under present-moment framing optimize for the present moment — short-term wins compress long-term costs into invisibility. The maintenance burden, dependency drift, and ownership transfer that determine whether a decision ages well are exactly the things that do not show up in the current discussion. Projecting forward in time forces those costs into the same plane as the present-moment benefits, so the comparison becomes honest.

**Detect when**:
- The solution looks obviously good in the short term but introduces dependencies, maintenance burden, or technical debt
- The decision involves third-party dependencies or external systems that could change
- The person making the decision now will not be the one paying the long-term cost

**Inject this question**:
> What new problems will this decision create three years from now? Who will pay that cost?

**Skip when**: The user has already discussed long-term tradeoffs or explicitly acknowledged the maintenance cost.

---

## 6. Cross-Domain Analogy

**Why it matters**: Many problems — coordination, queuing, caching, trust, incentive design, rate limiting — have been solved repeatedly in domains the user does not work in. Without a deliberate prompt to look outside, the option space stays bounded by the user's own field, and incremental variants of familiar approaches dominate. Pulling in an analogy from a distant domain rarely gives the final answer, but it reliably introduces a structurally different option that would not otherwise appear.

**Detect when**:
- All options feel incremental — no genuinely different directions are on the table
- The problem belongs to a known category (coordination, queuing, caching, trust, incentive design, rate limiting, etc.) that other domains have solved
- The user's frame of reference is bounded by a single field, limiting exposure to outside approaches

**Inject this question**:
> How has this problem been solved in other domains (logistics, finance, gaming, healthcare...)? What can their approach teach us?

**Skip when**: The user has already drawn on analogies from outside their domain.
