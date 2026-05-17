---
name: statusline-setup
description: Configure the Claude Code statusline — the persistent info bar shown below the input box. Use this skill when the user asks to "set up the statusline", "create a statusline", "configure the statusline", "add a statusline", "show model/branch/context in the status bar", or wants to display session context (model name, git branch, context window usage, quota) in Claude Code. Also trigger when the user asks to update, modify, or fix an existing statusline script.
---

# Statusline Setup

The statusline is a single line rendered below the Claude Code input box, refreshed on a configurable interval. It is driven by a shell command whose stdout becomes the displayed text. ANSI escape codes for color and bold/dim are supported.

## How It Works

Two pieces are needed:

1. **`~/.claude/statusline.sh`** — the script that prints the status line
2. **`~/.claude/settings.json`** — the `statusLine` top-level key that wires the script in

The script receives a JSON payload on stdin each time it is invoked. That payload contains session context Claude Code provides.

## Input JSON Schema

Key fields available from stdin:

```json
{
  "model": "claude-sonnet-4-6",
  "effort": "high",
  "cwd": "/path/to/project",
  "worktree": { "branch": "feat/my-feature" },
  "context_window": { "used_percentage": 42.7 },
  "quota_usage": { "percentage": 18.0 }
}
```

- `model` — model ID string (or object with `display_name`)
- `effort` — effort level string, or object with `.level`; may be absent
- `cwd` — current working directory
- `worktree.branch` — git branch from the active worktree (preferred over shell `git`)
- `context_window.used_percentage` — context fill 0–100
- `quota_usage.percentage` — plan quota used 0–100; absent on API key plans

## Standard Script

Create `~/.claude/statusline.sh` with this content:

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)

# folder
CWD_RAW=$(printf '%s' "$INPUT" | jq -r '.cwd // empty' 2>/dev/null)
[ -z "$CWD_RAW" ] && CWD_RAW="$PWD"
CWD=$(basename "$CWD_RAW")

# git branch
WORKTREE_BRANCH=$(printf '%s' "$INPUT" | jq -r '.worktree.branch // empty' 2>/dev/null)
if [ -n "$WORKTREE_BRANCH" ]; then
    BRANCH="$WORKTREE_BRANCH"
else
    BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "no-git")
fi
[ "${#BRANCH}" -gt 30 ] && BRANCH="${BRANCH:0:27}..."

# model
MODEL=$(printf '%s' "$INPUT" | jq -r '(if (.model | type) == "object" then .model.display_name // .model.id else .model end) // "unknown" | ltrimstr("claude-") | ltrimstr("Claude ")')
EFFORT=$(printf '%s' "$INPUT" | jq -r 'if .effort | type == "object" then .effort.level // "" else .effort // "" end')
if [ -n "$EFFORT" ]; then
    MODEL_STR="${MODEL} (${EFFORT})"
else
    MODEL_STR="${MODEL}"
fi

# context percentage
CTX_RAW=$(printf '%s' "$INPUT" | jq -r '.context_window.used_percentage // 0')
CTX=$(printf '%s' "$CTX_RAW" | awk '{printf "%d", int($1 + 0.5)}')
CTX=$(( CTX < 0 ? 0 : CTX > 100 ? 100 : CTX ))

# quota percentage (absent on API key plans)
QUOTA_RAW=$(printf '%s' "$INPUT" | jq -r '.quota_usage.percentage // empty' 2>/dev/null)
if [ -n "$QUOTA_RAW" ]; then
    QUOTA=$(printf '%s' "$QUOTA_RAW" | awk '{printf "%d", int($1 + 0.5)}')
    QUOTA=$(( QUOTA < 0 ? 0 : QUOTA > 100 ? QUOTA : QUOTA ))
    QUOTA_STR=" quota ${QUOTA}%"
else
    QUOTA_STR=""
fi

RESET="\033[0m"; BOLD="\033[1m"; DIM="\033[2m"
GREEN="\033[32m"; YELLOW="\033[33m"; RED="\033[31m"

pct_color() {
    local p=$1
    if   (( p < 50 )); then printf '%s' "$GREEN"
    elif (( p < 80 )); then printf '%s' "$YELLOW"
    else                    printf '%s' "$RED"
    fi
}

BAR_WIDTH=15
filled=$(( CTX * BAR_WIDTH / 100 ))
empty=$(( BAR_WIDTH - filled ))
bar=""
for (( i=0; i<filled; i++ )); do bar+="█"; done
for (( i=0; i<empty;  i++ )); do bar+="░"; done

CC=$(pct_color "$CTX")

printf " ${DIM}📁${RESET} ${CWD}  ${DIM}⎇${RESET} ${BRANCH}  ${DIM}◈${RESET} ${BOLD}${MODEL_STR}${RESET}  ${DIM}ctx${RESET} ${CC}[${bar}]${RESET} ${CC}${CTX}%%${RESET}${QUOTA_STR}\n"
```

Make it executable:
```bash
chmod +x ~/.claude/statusline.sh
```

## settings.json Configuration

Add the `statusLine` key at the **top level** of `~/.claude/settings.json`. It must NOT be nested under `hooks` — that is a common mistake.

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline.sh",
    "refreshInterval": 5
  }
}
```

- `type`: always `"command"`
- `command`: the shell command to run; receives JSON on stdin
- `refreshInterval`: seconds between refreshes (5 is a good default)

If `settings.json` already has other keys, merge `statusLine` in — do not overwrite the whole file.

## Smoke Test

After creating the script, verify it produces output before restarting Claude:

```bash
echo '{"model":"claude-sonnet-4-6","effort":"high","context_window":{"used_percentage":42}}' \
  | bash ~/.claude/statusline.sh
```

Expected output (with ANSI colors):
```
 📁 ~  ⎇ main  ◈ sonnet-4-6 (high)  ctx [██████░░░░░░░░░] 42%
```

## Customization Tips

- **Remove the folder segment**: delete the `CWD` lines and its `printf` token
- **Widen/narrow the bar**: change `BAR_WIDTH=15`
- **Change color thresholds**: edit the `pct_color` function cutoffs (50 / 80)
- **Suppress quota**: remove the `QUOTA_STR` logic and its `${QUOTA_STR}` reference
- **Shorten the branch**: change the `30`/`27` truncation constants

## Implementation Steps

When executing this skill:

1. Read `~/.claude/statusline.sh` if it exists — ask the user if they want to replace or update it
2. Write (or update) `~/.claude/statusline.sh` with the script above (or a customized version per user request)
3. Run `chmod +x ~/.claude/statusline.sh`
4. Read `~/.claude/settings.json` — merge in the `statusLine` block without removing other keys
5. Run the smoke test to confirm output
6. Tell the user the statusline will appear automatically — no restart needed
