---
name: end-session
description: End the current session — update context files, handle quest-specific bookkeeping, commit, and print summary
argument-hint: ""
user-invocable: true
---

# End Session

Wrap up the current working session. Auto-detect the quest type by reviewing the conversation history for the session start banner or `/start-session` invocation.

## Steps

### 1. Detect Quest Type and Ticket

Look at the beginning of this conversation for:
- A `SESSION START - MAIN QUEST` banner -> this was a **main** session
- A `SESSION START - SIDE QUEST` banner -> this was a **side** session
- If neither is found, assume **main**.

Also look for a `Ticket: <ref>` line in the session banner (e.g., `Ticket: GAM-5`). If found, carry this ticket reference through the end-of-session outputs. If no ticket was set, treat as "none".

### 2. Context Hygiene Check

Before wrapping up:
- **Review any corrections the user made during the session.** If any represent reusable preferences or patterns, propose adding them to `CLAUDE.md` (user-level or project-level as appropriate).
- **Check for discovered patterns**: build quirks, naming conventions, gotchas, or workflow preferences that emerged. Offer to persist these.

### 3. Adversarial Self-Evaluation

Before updating context files or declaring anything "accomplished," run a critical evaluation of the session's work. This combats the tendency to self-approve mediocre or incomplete work.

**Step 3a: Check against Definition of Done**
- Review the acceptance criteria defined at session start (from `SCRATCHPAD.md`, TodoWrite, or conversation history).
- For each criterion, honestly assess: **met**, **partially met**, or **not met**.
- Do NOT talk yourself out of issues you identify. If something is mediocre, say so.

**Step 3b: Project Health Check**
- Check if the project's `CLAUDE.md` documents a build command, test command, or lint command.
- If a **build command** exists, run it. If it fails, the session has introduced a regression — flag it as a blocker.
- If a **test command** exists, run it. Report failures — don't commit code that breaks existing tests.
- If a **lint command** exists, run it. Note warnings but don't block on them.
- If no commands are documented, skip silently — but note in the summary that health checks were unavailable.
- Include the results in the session summary (e.g., "Health: build passed, 47 tests passed, 2 lint warnings").

**Step 3c: Interact with the output**
- Where possible, actually test what was built — run the app, execute the code, hit the endpoints, check the UI.
- Don't just read the code and assume it works. Use available tools (terminal, browser, MCP servers) to verify.
- If you can't test (e.g., no dev server configured), explicitly note this as a gap.

**Step 3d: Grade the session honestly**
Categorize each deliverable into one of these statuses:
- **Done & Verified**: tested, working, meets the acceptance criteria
- **Done but Unverified**: code written but not tested or interacted with
- **Partial**: started but not complete — be specific about what's missing
- **Skipped**: planned but didn't get to it — note why

Use these categories in the session summary (Step 10) instead of a flat "Accomplished" list.

### 4. Update Context Files

#### `NEXT_SESSION.md`
Rewrite this file with fresh continuity notes for whoever picks up next:
- **Ticket worked on** (if a ticket was set this session, include it prominently at the top — e.g., `Ticket: GAM-5`)
- What was accomplished this session (use the graded statuses from Step 3d)
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

### 5. Merge Feature Branch (if applicable)

If the session's work was done on a feature branch AND the session goals are complete:
- Confirm all tests pass on the feature branch
- Merge to main with `--no-ff` and a descriptive merge commit
- Delete the feature branch after merge
- If the session goals are NOT complete (partial work, blocked, etc.), skip this step and note the branch status in the session summary

### 5b. Clean Up Stale Branches

Worktree subagents and previous sessions can leave behind stale branches. Clean them up:

1. **Prune stale remote tracking refs:** `git remote prune origin`
2. **Delete merged local branches** (excluding `main`): `git branch --merged main | grep -v '^\*\|main' | xargs -r git branch -d`
3. **Delete merged remote branches** (excluding `main`): `git branch -r --merged main | grep -v 'main\|HEAD' | sed 's|origin/||' | xargs -r -I{} git push origin --delete {}`
4. **Report any unmerged branches** to the user — don't delete them without approval

Include the count of deleted branches in the session summary (e.g., "Cleaned up: 5 stale branches").

### 6. Docker Container Rebuild (if applicable)

If the project uses Docker, auto-rebuild containers when session changes affect them.

**Step 6a: Detect Docker infrastructure**
- Check if `docker-compose.yml`, `docker-compose.yaml`, `compose.yml`, `compose.yaml`, or any `Dockerfile` exists in the project root or common subdirectories.
- If none found, skip this step entirely (no output needed).

**Step 6b: Check if changes affect containers**
- Compare files modified during this session (from `git diff` and `git status`) against container-affecting files:
  - `Dockerfile*`, `docker-compose*.yml`, `compose*.yml`
  - `requirements.txt`, `pyproject.toml`, `poetry.lock`, `Pipfile`, `Pipfile.lock`
  - `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
  - `.env`, `.env.*` (but not `.env.example`)
  - Any file explicitly referenced in a `COPY` or `ADD` directive in a Dockerfile
  - Config files mounted as volumes in compose (check the `volumes:` section)
- If no container-affecting files were changed, skip the rebuild.

**Step 6c: Rebuild and restart**
- Run `docker compose down` to stop running containers for this project.
- Run `docker compose up -d --build` to rebuild images and restart containers.
- If the rebuild fails, report the error in the session summary but don't block the rest of the end-session flow.
- Include rebuild status in the session summary (e.g., "Docker: rebuilt 2 services" or "Docker: rebuild failed — see error above").

### 7. Re-apply Session Title

Re-append the session title to combat the 64KB eviction issue (see `rename-session` skill). Use the `rename-session` skill with the same title that was set at session start. This keeps the custom title near the end of the `.jsonl` file so it stays visible in the session list.

### 8. Commit and Push

- Stage all changed files (on whichever branch is current after step 4)
- Write a clear commit message summarizing the session's work
- Push to the current branch
- If commit or push fails, report the error — don't silently skip it

### 9. Suggest Fresh Start

If the session was long or involved multiple large changes, explicitly recommend starting a fresh session next time rather than continuing this one. Mention if `/compact` would help if continuing is preferred.

### 10. Print Session Summary

#### For Main Quest sessions:

```
======================================
  SESSION END - MAIN QUEST
  Ticket: <ticket ref or "none">
  ------------------------------------
  Done & Verified:
    * <item — tested and meets acceptance criteria>

  Done but Unverified:
    * <item — code written, not tested>

  Partial:
    * <item — what's missing>

  Skipped:
    * <item — why>

  Definition of Done met: <Y/N/Partial>
  Health: <build passed, N tests passed | build failed | no commands documented>
  Docker: <rebuilt N services | rebuild failed | no changes | N/A>
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
  Ticket: <ticket ref or "none">
  ------------------------------------
  Explored: <item name>

  Done & Verified:
    * <outcome — tested and working>

  Done but Unverified:
    * <outcome — not tested>

  Partial:
    * <outcome — what's missing>

  Status: <completed | partial - needs another session>
  Health: <build passed, N tests passed | build failed | no commands documented>
  Docker: <rebuilt N services | rebuild failed | no changes | N/A>
  Main quest impact: <none | describe if any>
  CLAUDE.md updates proposed: <Y/N>
======================================
```

## Notes

- This is a **generic** session end skill. Individual projects can override it with `.claude/skills/end-session/SKILL.md` in their repo to add project-specific steps (running tests, build verification, lint checks, changelog updates, etc.).
- Always update context files BEFORE committing so the commit includes the fresh state.
- If there are uncommitted changes that look unintentional (debug logs, temp files), ask the user before including them.
