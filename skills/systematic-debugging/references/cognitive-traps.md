# Cognitive Traps in Debugging

These are what actually make debugging hard. Technical complexity is secondary — psychology is primary.

Many of these traps already have corresponding observable triggers in the Failure Signals table in SKILL.md. This file names the underlying bias and its countermeasure, for cases where understanding the mechanism helps break out of a stuck loop.

| Trap | How it manifests | Countermeasure |
|---|---|---|
| **Confirmation bias** | You see evidence for your theory, ignore evidence against it | Actively try to DISPROVE your hypothesis, not prove it |
| **Anchoring** | You fixate on your first theory and bend all evidence to fit | After 2 failed experiments on the same theory, force yourself to generate 3 alternative explanations |
| **Proximity bias** | You blame the nearest code to the stack trace | Explicitly ask: "Is the symptom site the owning layer? What evidence do I have?" |
| **Sunk cost** | You keep a failing approach because you already invested hours | Time spent is gone. Ask: "What's the fastest path from HERE?" not "How do I salvage what I've done?" |
| **Availability bias** | You assume it's the same kind of bug you saw last week | Similar symptoms can have completely different causes. Check the evidence for THIS bug |
| **Assumption blindness** | You're certain something is true without verifying | The thing you're most confident about is often wrong. Print it. Check it. Verify. |
| **Theory-before-data** | You jump to "I bet it's X" before observing | Force yourself to state 3 observations before forming any hypothesis |
