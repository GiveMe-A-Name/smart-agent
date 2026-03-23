# Response Modes

Use the response mode that best improves the skill's judgment surface.

These are options, not a default template and not required sections.

## Common Modes

### 1. Diagnosis

Use when the user is unsure what is wrong.

Focus on:
- what the skill already teaches well
- what capability is missing or distorted
- which weakness matters most: trigger, boundary, invariants, decision signals, failure signals, or packaging

### 2. Targeted Rewrite

Use when one section is the bottleneck.

Focus on:
- naming the weakest section
- explaining why it teaches the wrong thing
- replacing only that section
- stating what behavior should change afterward

### 3. Capability Rewrite

Use when the whole file is organized around the wrong model.

Focus on:
- rebuilding around trigger, boundary, invariants, decision signals, and failure signals
- preserving good existing language where possible
- removing rigid SOP language unless it is essential to prevent a real failure mode

### 4. Package Map

Use when the file is bloated or the support split is unclear.

Focus on:
- what must stay inline as the skill's law
- what belongs in `references/`, `templates/`, `examples/`, or `scripts/`
- why the split improves decision quality instead of just shortening the file

### 5. Contrast Examples

Use when the real problem is weak judgment rather than missing prose.

Focus on:
- a good move versus a bad move
- boundary language versus routing language
- capability teaching versus fixed-template teaching

## Selection Heuristic

Ask:
1. what is the highest-impact weakness?
2. what response would correct that weakness with the least extra structure?
3. would this mode teach judgment, or merely prescribe a format?

If the response starts looking like a mandatory output template, step back and simplify.
