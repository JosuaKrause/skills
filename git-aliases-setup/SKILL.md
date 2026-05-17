---
name: git-aliases-setup
description: Set up the user's standard git aliases. Use this skill when the user asks to "set up git aliases", "configure git aliases", "add my git aliases", "restore git aliases", or is on a new machine and wants their git environment configured.
---

# Git Aliases Setup

Apply the user's standard global git aliases with:

```bash
git config --global alias.st "status -sb"
git config --global alias.nm "branch --no-merged"
git config --global alias.ci "commit"
git config --global alias.m "merge"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.ms "merge --no-ff --no-commit"
git config --global alias.cf "checkout -f"
git config --global alias.tree "log --graph --pretty=oneline --abbrev-commit"
git config --global alias.sup "submodule update --recursive --init"
```

## What each alias does

| Alias | Expands to | Purpose |
|-------|-----------|---------|
| `st` | `status -sb` | Short branch+status view |
| `nm` | `branch --no-merged` | List branches not yet merged |
| `ci` | `commit` | Commit shorthand |
| `m` | `merge` | Merge shorthand |
| `co` | `checkout` | Checkout shorthand |
| `br` | `branch` | Branch shorthand |
| `ms` | `merge --no-ff --no-commit` | Merge without fast-forward, staged but not committed |
| `cf` | `checkout -f` | Force checkout |
| `tree` | `log --graph --pretty=oneline --abbrev-commit` | Visual commit graph |
| `sup` | `submodule update --recursive --init` | Initialize and update all submodules |

## Implementation Steps

1. Run all `git config --global alias.*` commands above
2. Verify with `git config --global --list | grep alias`
3. Confirm all 10 aliases are present
