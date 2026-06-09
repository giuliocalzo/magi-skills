# skill-lab

A collection of [Cursor Agent Skills](https://docs.cursor.com) — reusable, model-agnostic workflows that the Cursor agent can read and follow on demand.

Each skill lives in its own directory and is defined by a `SKILL.md` file with YAML front matter (`name`, `description`) followed by the instructions the agent executes.

## Skills

| Skill | Description |
| --- | --- |
| [`magi-pr-review`](magi-pr-review/SKILL.md) | Review and fix pull requests using an Evangelion MAGI-style workflow with three independent model agents (ChatGPT, Opus, and Nemotron/Gemini), synthesizing findings via critical consensus before completion. |

## Usage

Skills are picked up automatically by the Cursor agent. To trigger one, describe the task in natural language — for example:

> Run a MAGI PR review on this pull request.

The agent matches the request against each skill's `description`, reads the relevant `SKILL.md`, and follows its workflow.

## Adding a skill

1. Create a new directory named after the skill (kebab-case).
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
