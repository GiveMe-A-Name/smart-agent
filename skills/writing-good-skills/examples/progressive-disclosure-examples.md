# Progressive Disclosure Examples

Use these examples when deciding what stays in `SKILL.md` or how to point to support files.

Each layer must work at its decision point:

- metadata supports discovery;
- `SKILL.md` supplies the core method for every invocation;
- support files add conditional knowledge, executable operations, or output material.

## Give Every Pointer A Read Condition

Weak:

```markdown
See `references/` for details.
```

The agent cannot tell whether reading more will help.

Strong:

```markdown
Read `references/api-errors.md` when the API returns a non-2xx response or the response schema differs from the documented success shape.
```

The condition is visible during execution and the file's purpose is explicit.

## Keep Unrecognized Gotchas Inline

Weak split:

```markdown
Read `references/query-rules.md` when special query handling is needed.
```

If the agent does not know that deleted users remain in the table, it cannot recognize that special handling is needed.

Strong main-file rule:

```markdown
The `users` table uses soft deletion. Include `deleted_at IS NULL` unless the user explicitly requests inactive accounts.
```

Move a long field map or schema to a reference, but keep the gotcha that determines whether to consult it in the main file.

## Choose The Resource By Runtime Job

- `references/`: facts or procedures the agent reads only for a known case.
- `scripts/`: deterministic or repeatedly reimplemented operations.
- `assets/`: templates and materials copied or transformed into outputs.
- `examples/`: contrasts used to calibrate a choice or boundary.

Do not duplicate the same rule across layers. The main file should remain usable when optional resources are never opened.

