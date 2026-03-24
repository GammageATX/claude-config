---
name: rename-session
description: Programmatically rename the current Claude Code session to a descriptive title
argument-hint: "<title>"
user-invocable: true
---

# Rename Session

Rename the current Claude Code conversation/session to a descriptive title. Works across environments by using the best available method.

## How It Works

Claude Code stores session data in `.jsonl` files. Appending a `custom-title` entry to the current session's file updates the title shown in the session list and sidebar.

## Steps

### 1. Determine the Title

- If `$ARGUMENTS` is provided, use it as the title.
- If no arguments, derive a title from context:
  - Check if `NEXT_SESSION.md` exists in the project root — combine project name + top priority
  - If no context files, summarize what the session is about from conversation history
  - Format: `<Project Name>: <Focus Area>` (e.g., "Golf Shot Tracker: Sprint 5 Practice Plan Generator")

### 2. Find the Session File

The current session's `.jsonl` file needs to be located. Run these commands:

```bash
# Find the Claude config directory
CLAUDE_DIR="${HOME}/.claude"

# The projects directory contains session files organized by project path
PROJECTS_DIR="${CLAUDE_DIR}/projects"

# Find the most recently modified .jsonl file (the active session)
SESSION_FILE=$(find "${PROJECTS_DIR}" -name "*.jsonl" -type f -printf '%T@ %p\n' 2>/dev/null | sort -rn | head -1 | cut -d' ' -f2-)
```

If `find -printf` isn't available (macOS), use:
```bash
SESSION_FILE=$(find "${PROJECTS_DIR}" -name "*.jsonl" -type f -exec stat -f '%m %N' {} \; 2>/dev/null | sort -rn | head -1 | cut -d' ' -f2-)
```

If neither works or `SESSION_FILE` is empty, try the Windows path:
```bash
CLAUDE_DIR="${APPDATA}/Claude"
# Then repeat the find
```

### 3. Append the Custom Title

```bash
echo '{"type":"custom-title","title":"'"${TITLE}"'"}' >> "${SESSION_FILE}"
```

### 4. Confirm

Print a brief confirmation:
```
Session renamed: "<TITLE>"
```

## Environment Notes

| Environment | Method | Behavior |
|-------------|--------|----------|
| **VS Code extension** | `.jsonl` append | Updates session list sidebar. Tab title updates on next session load. |
| **Claude Code CLI (terminal)** | `.jsonl` append | Updates `claude --resume` session list. |
| **Desktop app / Web** | First output line | No `.jsonl` access — rely on the first-line approach in `start-session`. The descriptive opening line determines the conversation title. |

### Known Limitation

On very long sessions (rare at session start), the `custom-title` entry can get pushed outside the 64KB tail window that the session list reads, causing the title to disappear. This is a [known Claude Code issue](https://github.com/anthropics/claude-code/issues/33165). Mitigations:

- **Re-rename at session end**: the `end-session` skill can re-append the title to keep it near the end of the file.
- **Keep titles short**: shorter titles are less likely to be evicted.

## Integration with start-session

This skill is called automatically by `start-session` after the user confirms their focus area. You can also invoke it manually at any time with `/rename-session My New Title`.

## Fallback Behavior

If the session file cannot be located (e.g., running in Cowork desktop mode or web), this skill will:
1. Print the title as the first output line (which the desktop/web app uses for naming)
2. Note that programmatic rename was not possible in this environment
