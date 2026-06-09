# magi-skills

![MAGI](magi.png)

A collection of **MAGI** skills for Cursor — Evangelion-style workflows that reach consensus across three independent model agents (MELCHIOR/ChatGPT, BALTHASAR/Opus, CASPER/Nemotron-or-Gemini). Each skill lives under `skills/{skill-name}/` and is defined by a `SKILL.md` file with YAML front matter (`name`, `description`) followed by the instructions the agent executes.

## Skills

| Skill | Description |
| --- | --- |
| [`magi-pr-review`](skills/magi-pr-review/SKILL.md) | Review and fix pull requests using an Evangelion MAGI-style workflow with three independent model agents (ChatGPT, Opus, and Nemotron/Gemini), synthesizing findings via critical consensus before completion. |
| [`magi-feature-build`](skills/magi-feature-build/SKILL.md) | Design and implement a new feature using the same MAGI-style workflow: three independent model agents produce competing designs, synthesize one plan, implement it, and run a critical consensus loop before completion. |

## Usage

Skills are picked up automatically by the Cursor agent. To trigger one, describe the task in natural language — for example:

> Run a MAGI PR review on this pull request.

The agent matches the request against each skill's `description`, reads the relevant `SKILL.md`, and follows its workflow.

## Installation

### Cursor Settings (Cursor 2.4+)

The simplest way to use skills. No server setup required.

1. Open Cursor Settings (`Cmd+Shift+J`)
2. Go to the **Rules** tab
3. Click **Add Rule** → **Remote Rule (GitHub)**
4. Enter: `https://github.com/giuliocalzo/magi-skills`
5. Select the skills you want to import

Skills are copied to your `.cursor/skills/` directory and automatically discovered.

## Adding a skill

1. Create a new directory named after the skill (kebab-case) under `skills/`.
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
