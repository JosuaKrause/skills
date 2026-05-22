---
name: sync-skill
description: Copy a skill between the skills repo and the global ~/.claude/skills/ folder. Use this skill when the user wants to install a skill, publish a skill, sync a skill, deploy a skill, copy a skill to the skills folder, or copy a skill from the skills folder to the repo.
---

# Sync Skill

Copies a skill directory between the skills repository and `~/.claude/skills/`.

## Paths

| Location | Path |
|---|---|
| Skills repo | `/Users/krause/workspace/skills/` |
| Global skills dir | `~/.claude/skills/` |

## Step 1 — Diff both sides

Always run this first, even if the user named a specific skill.

```bash
comm -23 \
  <(find /Users/krause/workspace/skills -maxdepth 2 -name "SKILL.md" -o -name "skill.md" | sed 's|/[Ss][Kk][Ii][Ll][Ll]\.md||' | xargs -I{} basename {} | sort) \
  <(ls ~/.claude/skills/ | sort)
```

```bash
comm -13 \
  <(find /Users/krause/workspace/skills -maxdepth 2 -name "SKILL.md" -o -name "skill.md" | sed 's|/[Ss][Kk][Ii][Ll][Ll]\.md||' | xargs -I{} basename {} | sort) \
  <(ls ~/.claude/skills/ | sort)
```

Report the results as:

- **Only in repo** (not installed globally) — candidates to install
- **Only in `~/.claude/skills/`** (not in repo) — candidates to save back
- **In both** — in sync

If the user asked for a status check only, stop here and summarize. Otherwise continue.

## Step 2 — Determine direction and skill name

If the user's message makes the intent clear (e.g. "install new-aks-app", "copy statusline-setup back to the repo"), infer the values and confirm. Otherwise ask:

1. **Direction**: repo → global ("install") or global → repo ("save back")?
2. **Skill name**: which skill slug? Use the diff output from Step 1 to suggest likely candidates.

To install all repo skills at once, proceed with every skill in the "only in repo" list.

## Step 3 — Copy

### Repo → Global (install)

```bash
cp -r /Users/krause/workspace/skills/SKILL_NAME/ ~/.claude/skills/SKILL_NAME/
```

If the copied directory contains `skill.md` (lowercase) instead of `SKILL.md`, rename it:

```bash
[ -f ~/.claude/skills/SKILL_NAME/skill.md ] && \
  mv ~/.claude/skills/SKILL_NAME/skill.md ~/.claude/skills/SKILL_NAME/SKILL.md
```

### Global → Repo (save back)

```bash
cp -r ~/.claude/skills/SKILL_NAME/ /Users/krause/workspace/skills/SKILL_NAME/
```

If the copied directory contains `skill.md` (lowercase), rename it to match the repo convention:

```bash
[ -f /Users/krause/workspace/skills/SKILL_NAME/skill.md ] && \
  mv /Users/krause/workspace/skills/SKILL_NAME/skill.md \
     /Users/krause/workspace/skills/SKILL_NAME/SKILL.md
```

## Step 4 — Verify

Confirm `SKILL.md` is present at the destination:

```bash
# Repo → Global
ls ~/.claude/skills/SKILL_NAME/

# Global → Repo
ls /Users/krause/workspace/skills/SKILL_NAME/
```

## Step 5 — Update README (save-back only)

When saving back to the repo, check whether the skill already has an entry in `/Users/krause/workspace/skills/README.md`. If not, add a row to the skills table with the slug and the first line of its `description` field.
