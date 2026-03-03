---
name: skill-fix
description: Applies fixes to skill definitions using available skills methodology. Use when: (1) Observer detects issues in skill quality, (2) Skill needs structural/metadata/content fixes, (3) Validating against skill standards. NOT for: agent fixes, new skill creation.
---

# Skill Fix

Apply fixes to skill definitions by dynamically loading and using available skills.

## Workflow

1. **Discover available skills**: Explore what skills can be loaded
   - Check `available_skills` in Skill tool
   - Look for related skills (writing-skills, coding-standards, etc.)

2. **Load relevant skills**: Use Skill tool to load applicable skills
   - Try loading: writing-skills, skill-creator, coding-standards
   - If unavailable, use built-in validation

3. **Read target**: Load the skill's SKILL.md

4. **Analyze issue**: Identify what's wrong (structure/metadata/content/resources/effectiveness)

5. **Apply fix**: Modify using edit/write with loaded skill guidance

6. **Validate**: Ensure meets skill standards:
   - YAML frontmatter valid with name + description
   - name < 64 chars, hyphen-case
   - description has WHAT + WHEN
   - Body < 500 lines, progressive disclosure applied

7. **Report**: Summarize changes and validation results

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
