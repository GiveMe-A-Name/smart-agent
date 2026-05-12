# smart-agent

`smart-agent` is an open repository of reusable agent skills, role definitions, and engineering judgment frameworks.

The project is organized around two primary concepts:

- `skills/`: modular skill packages that define when an agent should invoke a capability, what it is responsible for, and how success is judged
- `agents/`: agent role definitions that shape higher-level working style and decision-making behavior

Rather than being a conventional application repository, this project serves as a structured knowledge and prompt asset library for agent-driven software engineering workflows.

## Repository Structure

```text
.
├── agents/          # Agent role definitions
├── skills/          # Reusable skill packages
```

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
- `technical-communication`: writing technical documents that are clear and actionable
- `understanding-codebases`: building code-backed understanding before making changes
- `code-review`: performing structured review of code changes

## Getting Started

If you are new to the repository, a good reading order is:

1. `skills/understanding-codebases/SKILL.md`
2. `skills/implementation-judgment/SKILL.md`
3. `skills/technical-communication/SKILL.md`
4. `skills/systematic-debugging/SKILL.md`
5. `skills/code-review/SKILL.md`

This sequence gives a representative view of how the repository approaches investigation, implementation, communication, debugging, and review.

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

## Adding a New Skill

New skills generally follow the existing directory pattern:

```text
skills/<skill-name>/
├── SKILL.md
├── examples/
├── references/
└── templates/
```

At minimum, a new `SKILL.md` should define:

- trigger conditions
- capability boundaries
- invariants or operating constraints
- completion criteria
- common failure signals

## Use Cases

This repository is best suited for:

- maintaining a reusable set of agent skills
- encoding engineering judgment, not just tool usage instructions
- standardizing workflows for design, implementation, debugging, review, and documentation
- serving as a prompt and guidance library for agent-based development systems

## Project Status

At the top level, the repository does not currently expose a conventional application entrypoint, build pipeline, or package manager manifest. Based on the current structure, it should be understood as a curated repository of agent guidance assets rather than a directly runnable product.
