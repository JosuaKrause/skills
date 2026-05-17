# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository stores Claude Code skills — reusable instruction sets that Claude executes when invoked via `/skill-name` slash commands. Skills are installed globally from `~/.claude/skills/` and locally from `.claude/skills/` within a project.

## Structure

Each skill is a directory named after the skill slug, containing a single `SKILL.md` file:

```
<skill-name>/
  SKILL.md
```

## SKILL.md format

Every `SKILL.md` must start with YAML frontmatter followed by the skill body:

```markdown
---
name: skill-slug
description: One-line trigger description used by Claude to decide when to invoke this skill.
---

# Skill Title

...implementation instructions...
```

- `name`: must match the directory name exactly
- `description`: written as "Use this skill when the user asks to …" — this is what Claude reads to decide whether to invoke the skill

## Deploying skills

To make a skill available globally, copy its directory to `~/.claude/skills/`:

```bash
cp -r <skill-name>/ ~/.claude/skills/<skill-name>/
```

To make a skill available only within a specific project, copy it to that project's `.claude/skills/` directory.

## Adding a new skill

1. Create a directory named after the skill slug
2. Write `SKILL.md` with the frontmatter and implementation steps
3. Copy to `~/.claude/skills/` (or the target project's `.claude/skills/`)
