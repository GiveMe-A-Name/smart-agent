---
name: receiving-code-review
description: "Evaluate review feedback before acting on it — assess correctness against design intent, not blind compliance. TRIGGER when: PR comments, inline annotations, or pasted review suggestions exist and the task is deciding what to do. DO NOT TRIGGER when: the task is evaluating someone else's code changes rather than deciding how to respond to feedback on a change you are responsible for handling, or when no review feedback exists yet."
---

# Receiving Code Review

Evaluate feedback against three evidence sources: explicit intent sources, code evidence, and architectural constraints — not reviewer authority or social compliance. Explicit intent sources include the current conversation, PR description, issue, design doc, commit message, code comments, tests, or prior user/maintainer decisions. If no intent source is available, treat the intent as unknown and clarify instead of inventing it. Collaboration does not mean compliance.

Examples for applying this standard:
- If a reviewer says "extract this for testability," first grep for call sites and tests that would use the extraction. If no usage evidence exists, the claimed benefit is hypothetical; push back or clarify instead of adding abstraction.
- If a reviewer says "this should live in middleware," first locate the current error-handling contract and caller expectations. If middleware would change domain-specific errors into generic errors, surface that constraint before accepting.
- If a reviewer says "this does not support X," split the claim: verify whether the code supports X, then verify whether X is an explicit goal from an intent source. If X is not an explicit goal, clarify before changing the design toward X.

Every reviewer comment contains three separable claims: an **observation** (what the current code does), a **premise** (what goal the code should serve), and a **proposed change**. Verify each independently — the observation may be correct while the premise is wrong.

## Trigger Logic

**Invocation default**: Implementing review feedback without evaluation leads to blind compliance — applying suggestions that are technically wrong for this codebase, breaking existing behavior, undermining design intent, or gold-plating things that were never needed. The cost of invoking this skill on feedback that turns out to be straightforward is a short verification pass. The cost of skipping it is a wrong implementation that passes review but breaks in production or silently degrades the architecture.

Use when review feedback exists and the task is deciding what to do with it before touching code. The source does not matter — user-pasted comments, PR threads, screenshots, inline annotations, or structured output from a reviewer sub-agent all qualify.

Do not use when:
- no review feedback exists yet — the task is to write code, not evaluate feedback [because evaluating feedback that doesn't exist collapses into design speculation, which belongs to a different task]
- feedback has already been fully evaluated and accepted, and the only remaining step is mechanical implementation with no judgment remaining [because if evaluation is done and nothing remains except applying the decision, this skill has already completed — re-invoking it adds overhead to an already-resolved question]

## Capability Boundary

This skill owns the evaluation and response to review feedback. It produces a decision per feedback item: implement, push back, or clarify — with technical reasoning for each.

It does NOT:
- Request or dispatch code reviews
- Verify that implementation is correct after making changes
- Write the implementation itself
- Decide how accepted changes should land structurally — evaluation ends at whether and what to change, not at how to structure the change once accepted

## How to Read This Skill

Each rule below is a guard against a specific failure mode, stated in [brackets]. The purpose is primary — the rule is a heuristic for when that purpose applies. Before invoking a rule, check whether its stated failure mode is present in your situation. When a rule produces an obviously wrong result, re-read the [because] and ask whether the purpose applies here — that is the actual check, not surface compliance with the rule's phrasing.

## Invariants

**Verify before implementing.** Never implement a suggestion without first checking it is technically sound for this codebase and aligned with an explicit intent source. [Because a reviewer may see only the diff, while the repository and conversation contain constraints, prior decisions, and tradeoffs that shaped the approach. Their characterization of what the code should do may be wrong even when their observation about what the code currently does is correct. Implementing before verifying conflates two independent claims.]

**Intent must be evidenced, not invented.** Never accept a reviewer's characterization of what the code should achieve, or what goal it was built toward, without tracing it to an explicit intent source. If you wrote the change in the current conversation, that conversation is an intent source; if you inherited the change or context was compacted, locate another source or mark intent unknown. [Because the reviewer's framing is secondary evidence. Letting secondary evidence overwrite explicit intent sources — or inventing intent when no source exists — is how correct implementations get undone by well-meaning but incomplete feedback.]

**No performative agreement.** Never say "You're absolutely right!", "Great point!", "Thanks for catching that!", or any expression of gratitude. Actions speak — fix it and show the change. [Because agreement-signaling without evaluation is noise that masks whether the feedback was actually assessed. If the reviewer is right, show the fix. If they are wrong, state why. Anything in between obscures the actual judgment.]

**Clarify before implementing dependent items.** For each item in multi-item feedback, check whether implementing it depends on the outcome of any unclear item. An item is dependent if the clarification's outcome would change where the fix lands, what interface it touches, or whether the fix applies at all. If dependent → stop and clarify first. If independently correct under all possible clarification outcomes → proceed, but state the independence explicitly. [The actual risk is churn: fixing item 3 then discovering item 1 requires rewriting it. The guard against churn is dependency tracking, not blanket suspension. A bug fix that holds under any architectural outcome has no dependency on the architectural clarification.]


## Completion Criteria

- [ ] Each feedback item has one explicit decision: Accept, Push back, or Clarify.
- [ ] Each non-mechanical Accept cites the code, test, or usage evidence that makes the suggestion technically correct.
- [ ] Each design or architecture comment cites the explicit intent source used to evaluate it; if no intent source was available, the decision is Clarify or explicitly states intent unknown.
- [ ] Each Push back states the reviewer's premise, the contradicting evidence or constraint, and the current intent source when design intent is involved.
- [ ] Each Clarify asks a specific question that names the competing interpretations and what implementation choice depends on the answer.
- [ ] No response contains performative agreement or gratitude phrases such as "you're right," "great point," or "thanks".
- [ ] If one feedback item depends on another unresolved item, the dependent item is marked blocked/dependent rather than implemented.
- [ ] If feedback came from multiple reviewers, contradictions were surfaced instead of being silently resolved; if a synthesis path was possible, it was attempted before defaulting to escalation; severity was triaged before processing linearly.

**If any criterion is not met, return to the relevant section before exiting.**

## Verification Approach

This skill is artifact-type: the output is a set of decisions (Accept / Push back / Clarify) with technical reasoning per feedback item. Completion is verified by self-checking the decisions against the Completion Criteria checklist. The evidence is the per-item decision, cited code or test evidence, and cited intent source when design intent is involved — not the agent's impression that the feedback was carefully considered.

A response that addresses every comment is not necessarily complete. The underlying test: for each item, can the decision be traced to code evidence and an explicit intent source when intent matters — not just to the reviewer's framing?

## Anti-Rationalization Check

Pause before exiting.

Do not treat this section as another checklist to clear. Use it to challenge whether the feedback was genuinely evaluated.

This check exists because responses that look evaluated are not the same as responses that are evaluated. A decision of "Accept" can be written for any comment — it does not prove the code was read, the premise was checked, or the implementation intent was traced. This section externalizes that checkpoint.

Did I ignore any failure signals because a convenient resolution path already existed?

Am I exiting because feedback is genuinely evaluated and addressed, or because the current responses look coherent enough?

Did I accept any reviewer's framing of what the code should do without verifying it against an explicit intent source?

Completion-faking signals specific to receiving code review — stop if any apply:

- A comment was marked "Accept" without opening the referenced code and tracing the reviewer's premise to an explicit intent source when intent matters — the acceptance is performative, not verified
- A pushback was written using "I prefer it this way" without technical evidence — this is a preference dressed as a reasoned position, which is both ineffective and inaccurate
- The reviewer named what the goal should be, and that framing was accepted without tracing it to an explicit intent source — the reviewer may have named the goal wrong, or named a goal that was explicitly rejected during design
- Multiple reviewers converged on the same issue, and convergence was treated as proof — convergence shifts the prior but does not replace the verification requirement; multiple reviewers can share the same blind spot
- Every comment has a response, so the review looks complete — but items were not triaged by severity first, meaning high-severity design concerns may have been processed after low-severity style fixes they might have invalidated

## Decision Signals

Before applying these signals, evaluate the comment against the five dimensions in `references/evaluation-criteria.md`: problem vs. preference, consequence test, evidence first, scope classification, and value assessment.

These questions shape judgment for each feedback item. They are not sequential steps — the signal that matters depends on the comment.

**Reviewer's premise vs. reviewer's observation**: A comment like "this doesn't achieve X" contains two claims: (1) the current code doesn't do X, and (2) X was the correct goal. Verify claim 1 against the code. Verify claim 2 against an explicit intent source — not against what the reviewer says the goal is. If X was never explicitly confirmed, treat it as an unverified goal before building toward it. This is critical when X would alter global strategy, close existing capability paths, or conflict with capabilities added since the last explicit agreement.

**What kind of claim is this?**
- Bug / logic error / security → verify against the codebase; likely valid regardless of source
- Design / architecture suggestion → evaluate against design intent; a different approach is not automatically better
- Style / preference → author's call unless the team style guide mandates otherwise
- Over-engineering (generality, future-proofing, premature abstraction) → grep for actual usage before accepting; if nothing calls it, consider removal
- Reviewer confusion → improve code clarity regardless of whether the specific suggestion is right; confusion is a signal about the code, not only about the reviewer

**Does the comment account for visible constraints?**
- Does the comment address constraints visible in the code, tests, PR description, issue, current conversation, or prior maintainer decision?
- Does the comment apply a pattern that conflicts with a constraint in this codebase?
- Does the comment rely only on the local diff while an explicit intent source outside the diff changes the conclusion?

**Source calibration:** Human partner means the user or maintainer who can decide design intent. A human partner may carry architectural and intent context; a dispatched reviewer sub-agent may miss intent and history; an external reviewer may lack platform constraints. Verify before implementing regardless of source. If no person with intent authority is reachable and intent is unclear, mark the item Clarify instead of assuming the intent. A technically wrong suggestion is wrong regardless of source.

**Does this serve the original design goals, or a different set of goals?** A suggestion can be technically valid and still wrong for this code if it sacrifices a tradeoff the design intentionally made. Before accepting: name what dimension this suggestion optimizes, and whether that dimension was more important than what it costs.

**Verify usage claims with evidence, not arguments.** "Supports testability," "cleaner," "more extensible" are arguments. Grep before accepting claims about what depends on something, what would benefit from an abstraction, or what actual usage patterns exist. If nothing calls the suggested abstraction, flag removal rather than implementing it.


## Decision Heuristics

For each comment, reach one of three decisions:

**Accept and implement** when: the suggestion is technically correct AND aligned with design intent; it identifies a genuine issue regardless of design (bugs, security); or it improves code clarity without changing behavior.

For accepted changes that are non-trivial — structural changes, responsibility movement, abstraction introduction, or behavioral modification — decide how the change should land structurally before writing code. The reviewer identified what to change; structural judgment decides how it should land. Skipping this step is how accepted review feedback introduces architectural damage. For purely mechanical fixes (typos, naming, simple bug fixes, formatting), proceed directly.

**Push back** when: a suggestion breaks existing functionality; violates YAGNI; is technically incorrect for this stack; conflicts with established design intent without proposing why the intent should change; undoes a deliberate tradeoff without offering a better one; has legacy or compatibility justification for the current approach; or conflicts with prior architectural decisions from the user or maintainer.

Use technical evidence — reference tests, code, or platform constraints. Never push back with just "I prefer it this way." State the design intent, the tradeoff, and the evidence. When the disagreement is architectural, involve the human partner. Technical facts and data overrule opinions and personal preferences.

**Clarify** when: you don't understand what the reviewer is suggesting; the suggestion is ambiguous between multiple interpretations; you need more context to evaluate; or implementing it would require changing the design intent and you want to confirm that's the reviewer's actual position.

Ask a specific question. "Can you clarify?" is useless. "Are you suggesting we move validation into the controller layer, or that we add a separate validation middleware? The former changes our error-handling contract with callers." is useful.


## Handling Multi-Reviewer Feedback

**Triage serves two purposes: priority ordering** prevents high-severity items from being processed after low-severity ones they could invalidate; **interaction detection** prevents churn from fixing an item that a pending design decision will require rewriting. Priority ordering requires multiple items spanning different severity levels or layers. Interaction detection requires items that could invalidate each other. If neither condition holds, proceed directly without a triage pass.

Triage before processing linearly. Priority order:
1. **Blocking / critical** — bugs, security, correctness
2. **Design-level (Layer 0–1)** — evaluate before lower-layer fixes; acceptance may invalidate downstream feedback
3. **Implementation (Layer 2–3)** — group related comments; multiple related issues may resolve with one structural fix
4. **Style / preference** — batch last; author's call unless mandated

Surface contradictions — do not silently pick one side. Reviewer disagreements exist at the solution level; resolution often exists at the goal level. When suggestions appear contradictory, check whether the underlying goals are compatible before defaulting to "list options and wait." If a synthesis path exists that satisfies both goals, propose it. If the goals genuinely conflict, state both positions, explain which aligns with design intent, and escalate for resolution. If design-level rework is accepted, state that lower-layer feedback will be addressed in the reworked code.

See `references/multi-reviewer.md` for extended handling of contradictions, overlapping feedback, and volume management.


## Response Language

See `references/response-language.md` for phrasing: correct feedback, pushback, if you were wrong, if the reviewer misunderstood, GitHub inline comment replies.


## Failure Signals

Stop and reorient if:
- You are about to write "you're right", "great point", "thanks", or any variant — delete it, state the fix instead
- You are implementing an item that depends on an unresolved clarification elsewhere in the feedback
- You are implementing a suggestion without having verified it against both the codebase and the design intent
- You are accepting a design suggestion without being able to articulate how it serves the original design goals
- You accepted a reviewer's characterization of what the goal is, or what the code should achieve, without tracing it to an explicit intent source — the reviewer's premise about the goal may be wrong even when their observation about the current code is correct
- You cannot verify a suggestion but are proceeding anyway — state the limitation instead
- You are implementing feedback that contradicts the design intent without explicitly acknowledging the intent change and getting confirmation
- A suggestion is technically wrong but social pressure makes direct pushback feel impossible — state the technical finding anyway. If you genuinely cannot raise it in context, flag it to your human partner with "Strange things are afoot at the Circle K." This is a signal to escalate, not permission to defer.
- You are about to implement non-trivial structural changes from review feedback without first deciding how they should land structurally
- Two reviewers gave contradictory suggestions and you silently picked one — without surfacing the conflict and without checking whether their underlying goals are compatible
- You are processing 20+ comments linearly instead of triaging by severity and layer first
- You are fixing style issues in code that design-level feedback may require rewriting

**When a failure signal fires, reorient on local context — not the whole review:** (1) re-read the design intent source — commit messages, PR description, prior conversation; (2) trace the specific claim against the explicit intent source; (3) grep for actual usage if the signal involves patterns, abstractions, or dependencies.
