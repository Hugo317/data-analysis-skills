---
name: skill-spec
description: Reference for the Claude Code SKILL.md format — frontmatter fields, naming and description rules, directory layout, and invocation modes. Use when writing, reviewing, validating, or generating skill files, including the output skill-builder produces.
---

# SKILL.md format

A skill is a directory containing `SKILL.md`, plus optional supporting files.

## Location

- Project skills: `.claude/skills/<skill-name>/SKILL.md` (checked in, available in this repo)
- Personal skills: `~/.claude/skills/<skill-name>/SKILL.md` (available everywhere)

The directory name is the skill's identity. It must match `name` in the frontmatter.

## Frontmatter

```yaml
---
name: <skill-name>
description: <what it does AND when to use it>
---
```

- `name`: lowercase kebab-case, matching the directory name. This is also the slash
  command (`/skill-spec`).
- `description`: the only thing Claude sees when deciding whether to invoke the skill,
  so it carries the whole trigger. State both what the skill does and the situations
  that should fire it, using words that appear in a user's actual request. A description
  that only names the capability ("Formats data") will not fire; one that names the
  occasion ("Use when the user asks to clean a CSV or normalize column names") will.
- `disable-model-invocation: true` (optional): only the user can trigger it via
  `/<name>`. Use for anything with side effects — deploys, releases, pushes.

## Body

The body is instructions addressed to Claude, loaded into context when the skill fires.

- Write imperatives ("Run X, then check Y"), not prose about the domain.
- Keep it focused; a skill that tries to cover several unrelated workflows fires at the
  wrong times. Split instead.
- Bundle long references as sibling files (`references/foo.md`, `scripts/bar.py`) and
  point to them by relative path so they load only when needed.
- `$ARGUMENTS` interpolates what the user typed after the slash command.

## Validating a skill

- Directory name == `name`, kebab-case.
- Frontmatter present, valid YAML, `name` and `description` both non-empty.
- `description` names a trigger condition, not just a capability.
- Referenced sibling files exist.
