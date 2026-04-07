---
name: dispatching-parallel-agents
description: "Dispatch independent tasks to concurrent agents. TRIGGER when: multiple tasks have non-overlapping files and no shared state, and parallelism saves meaningful time. DO NOT TRIGGER when: tasks share diagnosis, state, or files, or only one task exists."
---

# Dispatching Parallel Agents

Delegate independent tasks to isolated agents running concurrently. Each agent receives precisely constructed context — never your session history — and returns a focused result for you to integrate.

<HARD-GATE>
Do NOT dispatch agents until you have confirmed that each task is genuinely independent. Parallel dispatch on related problems wastes work and produces conflicting changes.
</HARD-GATE>

## Capability Boundary

This skill covers: identifying independent problem domains, scoping agent tasks, dispatching concurrently, and integrating results.

This skill does NOT own:
- Diagnosing what is broken (do that before dispatching)
- Choosing which agent type to use for a given task
- Merging conflicting changes if agents edited overlapping code

If you do not yet understand what is broken, diagnose first. Parallel dispatch is for execution, not exploration.

## Decision Signals

### Pre-Dispatch

**Independence test:** A task is independent if you can answer yes to all of the following:
- Could fixing this task leave the others unaffected? (If fixing A might fix B, investigate together first)
- Would each agent edit non-overlapping files and resources?
- Can you write a complete, self-contained prompt for each agent without referencing the others?
- Does no agent need another's output to begin?

If any answer is no, tasks are not independent. Use sequential agents or investigate together first.

**When parallel is the wrong choice:** Even when tasks pass the independence test, prefer a single agent if there are only 2 tasks and one is much faster — the coordination overhead may not be worth it. Prefer sequential if tasks are exploratory (you don't know the scope) or if a shared resource (same config file, same database) would serialize them anyway.

**Agent scoping:** Each agent prompt must contain exactly what the agent needs and nothing more:
- One problem domain — one test file, one subsystem, one bug
- All necessary context inline — error messages, test names, relevant file paths — never "see session context"
- Explicit constraints — what the agent must NOT change
- Specified output format — what you need back to integrate results

If you cannot write a self-contained prompt, the task is not scoped correctly.

### Post-Dispatch

**Verification judgment:** After agents return, do not assume fixes are compatible. Check: did any agents edit the same files? If yes, review those changes for conflicts before running the full suite. Spot-check at least one agent's work — agents can make systematic errors that look correct in isolation.

**Partial failure handling:** When one agent succeeds and another fails, do not treat them as a batch that either all succeeds or all fails. Instead:
- Assess whether the successful agent's changes are independently valid — can they be committed and verified on their own?
- If the successful change is independent and correct: keep it. Don't discard good work because a sibling failed.
- If the successful change depends on the failed agent's work (despite passing the independence test): hold it until the failed task is resolved. The independence test has a false-positive — the tasks were not truly independent.
- If the failed agent's task invalidates assumptions the successful agent made (e.g., discovered a shared root cause): evaluate whether the successful agent's changes are still correct given the new information.
- Default to caution: when unsure whether to keep a partial result, hold it and verify before committing.

**Discovery-based cancellation:** Sometimes one agent discovers information that invalidates the premise of another agent's task. When reviewing agent results, check:
- Did any agent discover a shared root cause that affects other tasks? If one agent found that the config file is corrupt and another is trying to fix symptoms of that corruption, the second agent's work is likely wasted.
- Did any agent's findings change the understanding of the problem in a way that makes other agents' scopes wrong? If so, the correct response is to stop, reassess, and potentially re-dispatch with updated context.
- If agents are still running and you receive early results that invalidate other agents' premises, cancel the invalidated agents rather than letting them complete wasted work.

**Budget and timeout awareness:** Dispatch the minimum number needed — 3-4 agents is reasonable; 10+ suggests decomposition is wrong. For exploratory tasks where scope is uncertain, prefer sequential — the first agent's findings should inform the second agent's scope. If an agent is taking significantly longer than expected, check on it rather than waiting indefinitely.

## Invariants

- Each agent gets its own isolated context — never pass session history or "you know what we discussed"
- One agent per independent problem domain — do not bundle unrelated problems into one agent
- Dispatch only after the independence test passes — not speculatively

## Completion Criteria

- [ ] All invariants were followed (isolated context, one agent per domain, passed independence test).
- [ ] Result compatibility was verified, including file conflicts where relevant.
- [ ] If partial failure occurred, successful changes were assessed for independent validity.
- [ ] Any agent findings that invalidate other agents' premises were checked and handled.

**If any criterion is not met, return to the relevant section before exiting.**

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the parallelization and integration really hold.

Did I ignore any failure signals because enough agents returned plausible results?

Am I exiting because parallel dispatch was appropriate and results are integrated, or because the current outputs feel close enough to done?

## Failure Signals

Stop and reassess if:
- You are dispatching because the problem is confusing, not because tasks are independent — diagnose first
- Agent scope is "fix all the tests" or "investigate the system" — too broad; split into one domain per agent
- You have no plan for integrating results — know how you will verify compatibility before dispatching
- One agent discovered a root cause that invalidates other agents' tasks, but you are still waiting for them to complete
- You are dispatching 5+ agents — reassess whether the decomposition is right
- A partial failure occurred and you are discarding the successful agent's valid work along with the failure

See `references/prompt-template.md` for a concrete example of a well-scoped agent prompt.

