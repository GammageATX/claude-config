---
name: parallel-worktrees
description: Use git worktrees for isolated parallel development — major features, risky refactors, or concurrent workstreams
argument-hint: ""
user-invocable: true
---

# Parallel Worktrees

Use this skill when making large, risky, or multi-track changes that benefit from branch isolation. Worktrees let you work on multiple branches simultaneously without stashing or switching.

## When to Use Worktrees

- **Major feature branches** that touch many files and could destabilize main
- **Risky refactors** where you want a clean rollback path
- **Parallel workstreams** — e.g., implementing a feature in one worktree while fixing bugs in another
- **Experimentation** — try an approach in isolation without polluting your working branch
- **Subagent isolation** — spawn a subagent with `isolation: "worktree"` so it works on a copy of the repo

## How It Works in Claude Code

### Option 1: Subagent with Worktree Isolation

The simplest approach. When launching a subagent via the Agent tool, set `isolation: "worktree"`:

```
Agent(
  description: "Implement auth module",
  prompt: "Build the authentication module with JWT support...",
  isolation: "worktree"
)
```

The subagent gets its own copy of the repo. If it makes changes, the worktree path and branch are returned. If it makes no changes, the worktree is auto-cleaned.

### Option 2: Manual Worktree Management

For hands-on control:

```bash
# Create a worktree for a feature branch
git worktree add ../feature-auth -b feature/auth

# Work in it
cd ../feature-auth
# ... make changes, commit ...

# When done, merge back
cd ../main-repo
git merge feature/auth

# Clean up
git worktree remove ../feature-auth
```

## Patterns

### Parallel Feature + Fix

When you need to ship a fix while a feature is in progress:

1. Main worktree: continue feature work
2. Spawn subagent with `isolation: "worktree"` for the hotfix
3. Hotfix agent commits, returns branch name
4. Merge hotfix to main, continue feature work

### Experimental Branch

Try a risky approach without risk:

1. Spawn subagent with worktree isolation
2. If the experiment works: merge the branch
3. If it fails: discard — main branch is untouched

### Multi-Agent Parallel Development

For large tasks with independent components:

1. Break the task into independent modules
2. Spawn one subagent per module, each with `isolation: "worktree"`
3. Each agent works in its own branch
4. Review and merge results sequentially

## Rules

- **Always name branches descriptively**: `feature/auth`, `fix/login-bug`, `experiment/new-parser`
- **Keep worktrees short-lived**: create, work, merge, clean up. Don't let them linger.
- **Don't create worktrees for trivial changes** — they add overhead. Use them for changes that touch 5+ files or carry risk.
- **Clean up after yourself**: `git worktree remove` and delete the branch when done.
- **Report back**: when a worktree subagent finishes, summarize what was done and what branch to review.
