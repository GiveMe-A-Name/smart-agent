# smart-agent

`smart-agent` is an open repository of reusable agent skills, role definitions, and engineering judgment frameworks.

The project is organized around two primary concepts:

- `skills/`: modular skill packages that define when an agent should invoke a capability, what it is responsible for, and how success is judged
- `agents/`: agent role definitions that shape higher-level working style and decision-making behavior

Rather than being a conventional application repository, this project serves as a structured knowledge and prompt asset library for agent-driven software engineering workflows.

## Repository Structure

```text
.
├── skills/          # Reusable skill packages
└── README.md        # Repository overview and maintainer entry point
```

Most maintained content lives under `skills/`. Each skill directory is intentionally small at the top level so that an agent can first read `SKILL.md`, then pull in only the examples, references, or templates required for the task.

## What Is a Skill?

A skill is a self-contained capability package, typically defined by `skills/<name>/SKILL.md`.
Each skill describes:

- when it should be triggered
- what problem space it owns
- what is outside its scope
- what constraints, invariants, and completion criteria apply

Some skills also include supporting materials such as:

- `examples/`: examples, edge cases, or contrast cases
- `references/`: supporting guidance and judgment frameworks
- `templates/`: reusable prompt or document templates

## Included Topics

The repository already contains skills covering common engineering workflows, including:

- `brainstorming`: turning early ideas into reviewable designs
- `implementation-judgment`: applying structural judgment before writing behavior-changing code
- `systematic-debugging`: finding root causes instead of patching symptoms
- `systematic-research`: validating external information through source-backed research
- `technical-communication`: writing technical documents that are clear and actionable
- `understanding-codebases`: building code-backed understanding before making changes
- `code-review`: performing structured review of code changes
- `receiving-code-review`: evaluating reviewer feedback before accepting or pushing back
- `reviewing-plans`: checking implementation plans for fit, risk, and executability
- `writing-good-skills`: designing compact, reusable skill instructions
- `writing-plans`: turning settled goals into scoped, verifiable execution plans

## Installation

This repository can be installed directly from GitHub with [`npx skills`](https://www.npmjs.com/package/skills).

Install the repository globally:

```bash
npx skills add GiveMe-A-Name/smart-agent -g
```

Preview the skills available in the repository:

```bash
npx skills add GiveMe-A-Name/smart-agent --list
```

Install a specific skill only:

```bash
npx skills add GiveMe-A-Name/smart-agent --skill implementation-judgment -g
```

Notes:

- `GiveMe-A-Name/smart-agent` is the GitHub shorthand source format supported by `npx skills`
- `-g` installs to the user-level skill directory instead of the current project
- `--list` shows discoverable skills without installing them
- `--skill <name>` installs only the named skill instead of the whole repository
