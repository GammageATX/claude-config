---
name: start-session
description: Start a new session with quest type routing — main quest (critical path) or side quest (backlog exploration)
argument-hint: "[main|side]"
user-invocable: true
---

# Start Session

Begin a new working session. Quest type is determined by `$ARGUMENTS` (defaults to `main` if omitted).

## Pre-Flight (All Quest Types)

1. **Read `CLAUDE.md`** (both user-level `~/.claude/CLAUDE.md` and project-level if present) to load preferences and workflow rules.
2. **Check environment**: confirm what tools, MCP servers, and plugins are available. If something expected is missing, flag it.
3. **Assess context health**: if resuming a long-running conversation, check if a `/compact` or fresh session would be beneficial. Use `/clear` if the conversation has gone off-track (clean slate). Use `/compact` if on-track but context is getting large (preserves summary). If so, recommend the appropriate one before proceeding.

## Quest Routing

### If `$ARGUMENTS` is `main` (or empty/omitted):

**You are on the Main Quest — critical path work.**

1. **Read context files** (skip any that don't exist yet):
   - `NEXT_SESSION.md` — continuity notes from the previous session
   - `TODO.md` — current task list and priorities
2. **Name the session** (MUST be your very first output): derive a descriptive one-liner from NEXT_SESSION.md combining the project name and top priority. Example: "Golf Shot Tracker: Sprint 5 Practice Plan Generator". This line determines the conversation title in desktop/web — never open with a generic phrase like "Starting new session."
3. **Identify the critical path**: summarize the top 1-3 priorities from these files.
4. **State your plan**: tell the user what you'll focus on and in what order. For multi-file changes, outline the plan before executing (Plan Mode).
5. **Get user confirmation**: ask the user to confirm or adjust the focus area before proceeding.
6. **Rename the session**: once the user confirms their focus, programmatically rename the session using the `rename-session` skill. Derive the title from the confirmed focus: `<Project Name>: <Confirmed Focus Area>`. This ensures the session title reflects the actual work, not just the initial suggestion.
7. **Define "Done" (contract negotiation)**: before any work begins, explicitly state what "done" looks like for this session. Write 3-5 concrete acceptance criteria — not vague goals. This prevents goalpost-moving mid-session (where the agent declares things done when they're not). If the session has a `SCRATCHPAD.md`, write the criteria there. Otherwise, state them inline and include them in the TodoWrite list. Example:
   - "User can upload an avatar and see it on their profile page"
   - "All new endpoints have passing integration tests"
   - "The dashboard loads in under 2 seconds with 100 records"
8. **Expand scope for ambitious tasks**: if the confirmed focus is a full feature, multi-sprint goal, or anything that would take more than a single session — don't just jump in. Expand the one-liner into a structured spec first (key components, interfaces, data flow, dependencies). This mirrors the planner agent pattern: a one-sentence prompt dramatically underscopes the work. Spend 5-10 minutes on spec expansion to avoid hours of rework.
9. **Identify parallelization opportunities**: if the plan has independent steps, note which ones can run concurrently via subagents or worktrees.
10. **Side quest capture rule**: if at any point during this session you notice an unrelated idea, improvement, or tangent worth exploring later, append it to `SIDE_QUESTS.md` under a `## Backlog` heading with a one-liner description and date. Do NOT pursue it — stay on the main quest.

Print a session banner (AFTER the descriptive opening line):

```
======================================
  SESSION START - MAIN QUEST
  Priorities: <top 1-3 items>
  Parallel opportunities: <if any>
======================================
```

### If `$ARGUMENTS` is `side`:

**You are on a Side Quest — exploratory work from the backlog.**

1. **Read `SIDE_QUESTS.md`**. If it doesn't exist or is empty, tell the user there are no side quests queued and offer to start a main session instead.
2. **Name the session** (MUST be your very first output): derive a descriptive one-liner combining the project name and the side quest topic. Example: "PDB Agents — Side Quest: Branch cleanup automation". This determines the conversation title in desktop/web.
3. **Pick one contained item** — choose something self-contained that won't disrupt main quest code paths. Prefer items that are interesting, low-risk, and completable in a single session.
4. **State what you picked and why.** Get user confirmation before starting.
5. **Rename the session**: once the user confirms the side quest choice, programmatically rename the session using the `rename-session` skill. Title format: `<Project Name> — Side Quest: <Chosen Item>`.
6. **Guardrail**: do NOT modify files that are on the critical path of the main quest unless the side quest specifically requires it and you get explicit user approval.

Print a session banner (AFTER the descriptive opening line):

```
======================================
  SESSION START - SIDE QUEST
  Exploring: <chosen item>
======================================
```

## Session Workflow Reminders

Once the session is underway, remember these rules from CLAUDE.md:

- **Status updates**: use TodoWrite for all multi-step tasks. Echo status lines to terminal as backup.
- **Background work**: run independent steps in parallel (`run_in_background`, subagents, worktrees).
- **Plan before executing**: for 3+ file changes, outline the plan first and get approval.
- **Context hygiene**: if the session gets large, suggest compacting or splitting.
- **Learn from corrections**: if the user corrects you, offer to update CLAUDE.md.
- **Monitor for context anxiety**: watch for these symptoms in yourself — rushing through steps, skipping planned work, giving shorter/terser responses, declaring things "done" when they're only partially complete, or wrapping up prematurely. If you notice any of these, flag it to the user immediately and suggest a context reset or compaction. Don't let the session quality degrade silently.
- **Check work against Definition of Done**: before moving on from any major task, explicitly check it against the acceptance criteria defined at session start. Don't self-approve — be honest about what's actually complete vs. what's "close enough."

## Notes

- This is a **generic** session start skill. Individual projects can override it with `.claude/skills/start-session/SKILL.md` in their repo to add project-specific steps (build checks, test commands, architecture rules, etc.).
- If project-level context files reference project-specific tooling you don't recognize, ask the user rather than guessing.
