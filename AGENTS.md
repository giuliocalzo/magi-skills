# AGENTS.md

Guidance for AI agents working in this repository.

## Repository purpose

`magi-skills` is a skills distribution repository — a collection of [Cursor Agent Skills](https://docs.cursor.com). Skills live under `skills/`, each in its own kebab-case directory defined by a `SKILL.md` file with YAML front matter (`name`, `description`) followed by the agent instructions.

## Commit conventions

All commits MUST follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

Format:

```
<type>(<scope>): <description>
```

- **`<scope>` is REQUIRED and MUST be the skill name** (the skill's directory / `name` in its front matter) for any change that adds or modifies a skill. Use the kebab-case skill name, e.g. `magi-pr-review`.
- For repo-wide changes that are not tied to a single skill, use a `repo` scope (e.g. `chore(repo): update gitignore`).
- Each commit MUST be scoped to a single skill. Do not mix changes to multiple skills in one commit — split them into one commit per skill.

### Allowed types

- `feat` — a new skill or new capability within a skill
- `fix` — a correction to an existing skill's behavior or instructions
- `docs` — documentation-only changes (README, CONTRIBUTING, comments)
- `refactor` — restructuring a skill without changing its behavior
- `chore` — tooling, metadata, or maintenance changes
- `style` — formatting/wording changes with no behavioral impact

### Examples

```
feat(<skill-name>): add a new capability to the skill
fix(<skill-name>): correct a step in the skill's workflow
docs(repo): add contributing guide
chore(repo): add gitignore and license
```

## Skill authoring rules

- A new skill MUST live under `skills/<skill-name>/` and include a `SKILL.md` with `name` and `description` front matter.
- The `name` in front matter MUST match the directory name.
- Keep `description` action-oriented and include phrases users are likely to say.
- When adding or removing a skill, update the skills table in `README.md` in the same commit.
- Keep edits narrowly scoped; do not refactor unrelated skills.
