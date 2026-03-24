---
name: init-project
description: Bootstrap a new project with the full dev workflow — shared config submodule, session management files, and .claude structure
argument-hint: "[project-name]"
user-invocable: true
---

# Init Project

Set up a new project with the standard dev workflow. This creates the full `.claude/` structure, links the shared config, and scaffolds session management files.

## What It Creates

```
project-root/
  .claude/
    shared/              ← git submodule → claude-config repo
    commands/
      start-session.md   ← project-specific (references shared)
      end-session.md     ← project-specific (references shared)
    skills/              ← empty, ready for project-specific overrides
  CLAUDE.md              ← project context file with shared reference
  TODO.md                ← task list scaffolded with Sprint 1
  NEXT_SESSION.md        ← continuity file (initial state)
  SIDE_QUESTS.md         ← backlog for captured ideas
  .gitignore             ← sensible defaults
```

## Steps

### 0. Check Working Directory

Before anything else, check if you have a valid working directory:

- **If no folder is open** (no workspace, blank VS Code window): STOP. Tell the user:
  ```
  I need a folder to work in. Please either:
  1. Create a new folder where you want the project to live and open it in VS Code
  2. Tell me the full path where you want the project created (e.g., "C:\Users\Michael\Google Drive\05 - Hobbies\MyNewProject")
  ```
  If the user provides a path, create the directory if it doesn't exist, then proceed.

- **If a folder IS open** but it's not empty: ask the user if they want to initialize the workflow in the existing project (skip scaffolding files that already exist) or if this is the wrong folder.

- **If a folder IS open and it's empty** (or only has a `.git`): perfect — proceed.

### 1. Gather Project Info

Ask the user (if not provided via `$ARGUMENTS`):
- **Project name**: what to call it (used in session naming, CLAUDE.md header, etc.). Default to the folder name if the user doesn't specify.
- **Brief description**: one sentence on what the project does
- **Stack**: language(s), framework(s), database — or "TBD" if exploring
- **End-state vision**: what does "done" look like? (used to frame Sprint 1)

### 2. Initialize Git (if needed)

```bash
git init  # only if not already a git repo
```

### 3. Add Shared Config Submodule

```bash
git submodule add https://github.com/GITHUB_USERNAME/claude-config.git .claude/shared
```

> **Note:** Replace `GITHUB_USERNAME` with the actual GitHub username. If the submodule add fails (repo not yet created, no network), create the `.claude/shared/` directory manually and tell the user to add the submodule later.

### 4. Create `.claude/commands/`

#### `start-session.md`

```markdown
Read `NEXT_SESSION.md` in the project root for context from the previous session. Then:

1. **Name the session (MUST be your very first output line):** Derive a descriptive one-liner from `NEXT_SESSION.md` combining the project name and top priority. Format: `<PROJECT_NAME>: <Focus Area>`. This determines the conversation title — never open with a generic phrase like "Starting new session."
2. Read TODO.md to understand current priorities
3. Give a brief morning-style overview: what's the current state, what's the priority, and suggest what to work on today
4. Ask what the user wants to focus on
```

#### `end-session.md`

```markdown
End-of-session protocol. Do ALL of these steps IN ORDER:

## 1. Update NEXT_SESSION.md
Rewrite `NEXT_SESSION.md` in the project root with:
- What was accomplished this session (brief bullet points)
- What the next priority is (from TODO.md)
- Key decisions or design context the next session needs
- Current system state (whatever's relevant for this project)

## 2. Update TODO.md
- Mark completed items
- Reorder priorities if they changed during the session
- Add any new items that came up

## 3. Commit & Push
- Check `git status` for any uncommitted changes
- Commit and push everything with a descriptive message
- Report the total commit count for this session

## 4. Session Summary
Print a final summary with: features built, files changed, next priority.
```

### 5. Create Session Management Files

#### `CLAUDE.md`

```markdown
> **Shared preferences:** Read `.claude/shared/CLAUDE.md` before proceeding. It contains user-wide preferences for build philosophy, frontend style, code style, communication style, and session management. Project-specific instructions below override shared preferences where they conflict.

# <PROJECT_NAME> — Project Context

## What This Project Is

<BRIEF_DESCRIPTION>

## Stack

<STACK_INFO>

## Key Design Decisions

(none yet — add as they're made)

## File Locations

- `CLAUDE.md` — This file (project context)
- `TODO.md` — Sprint task list and priorities
- `NEXT_SESSION.md` — Session continuity notes
- `SIDE_QUESTS.md` — Backlog of ideas to explore later

## Running the Project

(fill in as the project takes shape)
```

#### `TODO.md`

```markdown
# TODO — <PROJECT_NAME>

## Sprint 1: Foundation
- [ ] <First task derived from end-state vision>
- [ ] <Second task>
- [ ] <Third task>

## Backlog
(items that come up but aren't Sprint 1 priority)
```

#### `NEXT_SESSION.md`

```markdown
# Next Session — <PROJECT_NAME>

## Last Session
Project initialized. No work done yet.

## Current State
- Fresh repo, no code yet
- Stack: <STACK_INFO>

## Next Priority
Sprint 1, Task 1: <first task>

## Starting Prompt
Start with Sprint 1. Read TODO.md for the full task list.
```

#### `SIDE_QUESTS.md`

```markdown
# Side Quests — <PROJECT_NAME>

## Backlog
(capture ideas here during main quest sessions)

## Completed
(move completed side quests here with date and notes)
```

### 6. Create `.gitignore` (if none exists)

Include sensible defaults based on the stated stack. Always include:
```
.env
.env.local
*.db
node_modules/
__pycache__/
.DS_Store
Thumbs.db
```

### 7. Initial Commit

```bash
git add -A
git commit -m "Initial project setup with shared claude-config

Scaffolded: CLAUDE.md, TODO.md, NEXT_SESSION.md, SIDE_QUESTS.md,
.claude/ structure with shared config submodule.

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

### 8. Report

Print a summary:
```
Project "<PROJECT_NAME>" initialized!

  .claude/shared/    → linked to claude-config
  .claude/commands/  → start-session, end-session
  CLAUDE.md          → project context (edit this as you build)
  TODO.md            → Sprint 1 scaffolded with <N> tasks
  NEXT_SESSION.md    → ready for first session
  SIDE_QUESTS.md     → empty backlog

Run `/start-session` to begin your first session.
```

## Notes

- This skill creates **minimal** project-level commands and skills. As the project grows, the user can add project-specific overrides (build checks, test commands, lint steps) to `.claude/skills/start-session/SKILL.md` and `.claude/skills/end-session/SKILL.md`.
- If the user already has code in the directory, don't overwrite existing files — ask first.
- The `.claude/commands/` files are lightweight starters. Suggest upgrading to full `.claude/skills/` versions once the project has enough build tooling to warrant health checks.
