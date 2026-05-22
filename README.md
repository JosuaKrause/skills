# Skills

A collection of reusable [Claude Code](https://claude.ai/code) skills — instruction sets that Claude executes when invoked via slash commands.

## What is a skill?

A skill is a directory containing a `SKILL.md` file. Claude reads the frontmatter `description` to decide when to invoke the skill, and follows the body to execute it. Skills may also include a `templates/` directory with ready-to-copy files that the skill references.

```
<skill-name>/
  SKILL.md
  templates/   ← optional: files the skill copies into a target repo
```

## Skills in this repo

| Skill | Description |
|---|---|
| [`git-aliases-setup`](git-aliases-setup/SKILL.md) | Configure standard global git aliases |
| [`statusline-setup`](statusline-setup/SKILL.md) | Set up the Claude Code statusline (model, branch, context bar) |
| [`new-aks-app`](new-aks-app/SKILL.md) | Scaffold a new FastAPI + React/TypeScript app (or add AKS deployment, Redis, or persistent storage to an existing one) on the shared lingolou-aks cluster; includes ready-to-copy templates |
| [`sync-skill`](sync-skill/SKILL.md) | Copy a skill between this repo and `~/.claude/skills/` (install or save back) |

## Installation

Copy a skill directory to `~/.claude/skills/` to make it available in all Claude Code sessions:

```bash
cp -r git-aliases-setup/ ~/.claude/skills/git-aliases-setup/
```

Or copy to a project's `.claude/skills/` to scope it to that project only.

## Adding a new skill

1. Create a directory named after the skill slug
2. Write `SKILL.md` with this structure:

```markdown
---
name: skill-slug
description: Use this skill when the user asks to …
---

# Skill Title

Implementation steps…
```

3. Copy to `~/.claude/skills/` (global) or `.claude/skills/` (project-scoped)
