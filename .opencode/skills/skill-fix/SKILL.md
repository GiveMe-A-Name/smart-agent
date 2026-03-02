---
name: skill-fix
description: Applies fixes to skill definitions using skill-creator methodology.Use when: (1) Observer detects issues in skill quality, (2) Skill needs structural/metadata/content fixes, (3) Validation against skill-creator standards is needed. NOT for: agent fixes, new skill creation.
---

# Skill Fix

Apply fixes to skill definitions using skill-creator methodology.

## Workflow

1. **Load skill-creator**: Use Skill tool to reference best practices
2. **Read target**: Load the skill's SKILL.md
3. **Analyze issue**: Identify what's wrong (structure/metadata/content/resources/effectiveness)
4. **Apply fix**: Modify using edit/write with skill-creator principles
5. **Validate**: Ensure meets skill-creator requirements:
   - YAML frontmatter valid with name + description
   - name < 64 chars, hyphen-case
   - description has WHAT + WHEN
   - Body < 500 lines, progressive disclosure applied
6. **Report**: Summarize changes and validation results

## Fix Categories

| Category | Common Issues |
|----------|---------------|
| Structure | Missing SKILL.md, invalid YAML |
| Metadata | Vague description, missing WHEN |
| Content | Too verbose, no progressive disclosure |
| Resources | Broken references, missing scripts |
| Effectiveness | Wrong triggers, no value add |

## Validation Checklist

- [ ] YAML frontmatter valid (name + description)
- [ ] name < 64 chars, hyphen-case
- [ ] description includes WHAT + WHEN
- [ ] Body concise, progressive disclosure applied
- [ ] References linked from SKILL.md

## Output

Report: skill path, issue category, changes applied, validation results, status
