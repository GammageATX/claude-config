---
name: end-session
description: End the current session — update context files, handle quest-specific bookkeeping, commit, and print summary
argument-hint: ""
user-invocable: true
---

# End Session

Wrap up the current working session. Auto-detect the quest type by reviewing the conversation history for the session start banner or `/start-session` invocation.

## Steps

### 1. Detect Quest Type

Look at the beginning of this conversation for:
- A `SESSION START - MAIN QUEST` banner -> this was a **main** session
- A `SESSION START - SIDE QUEST` banner -> this was a **side** session
- If neither is found, assume **main**.

### 2. Context Hygiene Check

Before wrapping up:
- **Review any corrections the user made during the session.** If any represent reusable preferences or patterns, propose adding them to `CLAUDE.md` (user-level or project-level as appropriate).
- **Check for discovered patterns**: build quirks, naming conventions, gotchas, or workflow preferences that emerged. Offer to persist these.

### 3. Update Context Files

#### `NEXT_SESSION.md`
Rewrite this file with fresh continuity notes for whoever picks up next:
- What was accomplished this session
- What's in progress or partially done
- Any blockers, open questions, or decisions needed
- Key files that were modified
- **Recommended session type for next time** (main or side) and why

#### `TODO.md`
- Mark completed items as done (e.g., `- [x]`)
- Add any new tasks discovered during the session
- Re-prioritize if the session changed what matters most

#### `SIDE_QUESTS.md`
- **Main quest sessions**: if you captured any side quest ideas during the session, list them in the summary. Ensure they're in `SIDE_QUESTS.md` under `## Backlog` with date and one-liner.
- **Side quest sessions**: mark the explored item as completed with a brief note of what was built/learned. Move it from `## Backlog` to a `## Completed` section with the date.

### 4. Merge Feature Branch (if applicable)

If the session's work was done on a feature branch AND the session goals are complete:
- Confirm all tests pass on the feature branch
- Merge to main with `--no-ff` and a descriptive merge commit
- Delete the feature branch after merge
- If the session goals are NOT complete (partial work, blocked, etc.), skip this step and note the branch status in the session summary

### 4b. Clean Up Stale Branches

Worktree subagents and previous sessions can leave behind stale branches. Clean them up:

1. **Prune stale remote tracking refs:** `git remote prune origin`
2. **Delete merged local branches** (excluding `main`): `git branch --merged main | grep -v '^\*\|main' | xargs -r git branch -d`
3. **Delete merged remote branches** (excluding `main`): `git branch -r --merged main | grep -v 'main\|HEAD' | sed 's|origin/||' | xargs -r -I{} git push origin --delete {}`
4. **Report any unmerged branches** to the user — don't delete them without approval

Include the count of deleted branches in the session summary (e.g., "Cleaned up: 5 stale branches").

### 5. Re-apply Session Title

Re-append the session title to combat the 64KB eviction issue (see `rename-session` skill). Use the `rename-session` skill with the same title that was set at session start. This keeps the custom title near the end of the `.jsonl` file so it stays visible in the session list.

### 6. Commit and Push

- Stage all changed files (on whichever branch is current after step 4)
- Write a clear commit message summarizing the session's work
- Push to the current branch
- If commit or push fails, report the error — don't silently skip it

### 7. Suggest Fresh Start

If the session was long or involved multiple large changes, explicitly recommend starting a fresh session next time rather than continuing this one. Mention if `/compact` would help if continuing is preferred.

### 8. Print Session Summary

#### For Main Quest sessions:

```
======================================
  SESSION END - MAIN QUEST
  ------------------------------------
  Accomplished:
    * <item 1>
    * <item 2>

  CLAUDE.md updates proposed: <Y/N>
  Side quests captured: <N>
    * <brief descriptions if any>

  Next up: <top priority for next session>
  Recommendation: <fresh session | continue>
======================================
```

#### For Side Quest sessions:

```
======================================
  SESSION END - SIDE QUEST
  ------------------------------------
  Explored: <item name>

  What was built/learned:
    * <outcome 1>
    * <outcome 2>

  Status: <completed | partial - needs another session>
  Main quest impact: <none | describe if any>
  CLAUDE.md updates proposed: <Y/N>
======================================
```

## Notes

- This is a **generic** session end skill. Individual projects can override it with `.claude/skills/end-session/SKILL.md` in their repo to add project-specific steps (running tests, build verification, lint checks, changelog updates, etc.).
- Always update context files BEFORE committing so the commit includes the fresh state.
- If there are uncommitted changes that look unintentional (debug logs, temp files), ask the user before including them.
