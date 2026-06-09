# skill-lab

A collection of [Cursor Agent Skills](https://docs.cursor.com) — reusable, model-agnostic workflows that the Cursor agent can read and follow on demand.

Skills live under `.cursor/skills/`, the directory Cursor scans automatically. Each skill is its own folder containing a `SKILL.md` file with YAML front matter (`name`, `description`) followed by the instructions the agent executes.

## Skills

| Skill | Description |
| --- | --- |
| [`magi-pr-review`](.cursor/skills/magi-pr-review/SKILL.md) | Review and fix pull requests using an Evangelion MAGI-style workflow with three independent model agents (ChatGPT, Opus, and Nemotron/Gemini), synthesizing findings via critical consensus before completion. |

## Usage

Skills are picked up automatically by the Cursor agent. To trigger one, describe the task in natural language — for example:

> Run a MAGI PR review on this pull request.

The agent matches the request against each skill's `description`, reads the relevant `SKILL.md`, and follows its workflow.

## Installation

Cursor only discovers skills under `.cursor/skills/`. To use these skills in a project, make this repo's skills available at that path — for example, clone it and symlink:

```bash
git clone git@github.com:giuliocalzo/skill-lab.git
ln -s "$(pwd)/skill-lab/.cursor/skills" /path/to/your-project/.cursor/skills
```

Or copy the individual skill folders into your project's `.cursor/skills/` (or your personal `~/.cursor/skills/`).

## Adding a skill

1. Create a new directory named after the skill (kebab-case) under `.cursor/skills/`.
2. Add a `SKILL.md` with front matter:

```markdown
---
name: my-skill
description: One or two sentences describing what the skill does and when to use it.
---

# My Skill

Instructions the agent should follow...
```

3. Keep the `description` action-oriented and include the phrases users are likely to say, so the agent reliably selects the skill.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for guidelines on adding or improving skills.

## License

Released under the [WTFPL](LICENSE) — do what the fuck you want to.
