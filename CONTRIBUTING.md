# Contributing to skill-lab

Thanks for your interest in contributing! This repo is a collection of Cursor Agent Skills, so most contributions are either a new skill or an improvement to an existing one.

## Adding a new skill

1. Create a directory named after the skill in kebab-case under `.cursor/skills-cursor/` (e.g. `.cursor/skills-cursor/my-skill/`).
2. Add a `SKILL.md` with YAML front matter and instructions:

```markdown
---
name: my-skill
description: One or two sentences describing what the skill does and when to use it.
---

# My Skill

Instructions the agent should follow...
```

3. Write the `description` to be action-oriented and include the phrases users are likely to say, so the agent reliably selects the skill.
4. Add a row for the skill to the table in [`README.md`](README.md).

## Improving an existing skill

- Keep edits focused and explain the motivation in your commit message / PR description.
- Preserve the front matter contract (`name`, `description`).
- If behavior changes meaningfully, update the README description to match.

## Style guidelines

- Use clear, imperative instructions.
- Prefer model-agnostic wording unless a step genuinely requires a specific model.
- Keep skills self-contained and avoid assumptions about the user's environment.

## Submitting changes

1. Fork and create a feature branch.
2. Make your changes.
3. Open a pull request describing what changed and why.

## License

By contributing, you agree that your contributions will be licensed under the project's [WTFPL](LICENSE).
