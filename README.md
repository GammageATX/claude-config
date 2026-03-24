# claude-config

Shared Claude Code configuration for all of Michael's projects. This repo serves as the single source of truth for user preferences, workflow skills, and session management — accessible from any environment (home PC, work PC, Claude Code web).

## How It Works

This repo is added as a **git submodule** at `.claude/shared/` in each project. Project-level CLAUDE.md files include a directive to read shared preferences first, and project-level skills can override or extend the shared skills.

### Inheritance Model

```
claude-config/CLAUDE.md          (user preferences — always loaded)
  └── project/.claude/CLAUDE.md  (project-specific context — overrides/extends)

claude-config/skills/            (shared skills — generic versions)
  └── project/.claude/skills/    (project skills — override with project-specific steps)
```

## Integration

### Add to a new project

```bash
cd your-project
git submodule add https://github.com/YOUR_USERNAME/claude-config.git .claude/shared
git commit -m "Add shared claude-config as submodule"
```

Then add this line to the **top** of your project's `CLAUDE.md`:

```markdown
> **Shared preferences:** Read `.claude/shared/CLAUDE.md` before proceeding. It contains user-wide preferences for build philosophy, frontend style, code style, communication style, and session management. Project-specific instructions below override shared preferences where they conflict.
```

### Update shared config in all projects

```bash
cd your-project/.claude/shared
git pull origin main
cd ../..
git add .claude/shared
git commit -m "Update shared claude-config"
```

### Clone a project with the submodule

```bash
git clone --recurse-submodules https://github.com/YOUR_USERNAME/your-project.git
# OR if already cloned:
git submodule update --init --recursive
```

## Contents

| File | Purpose |
|------|---------|
| `CLAUDE.md` | User preferences: environment, workflow rules, session management, build philosophy, frontend/code style, communication style |
| `skills/start-session/` | Generic session start with quest routing (main/side) and conversation naming |
| `skills/end-session/` | Generic session end with context file updates, branch cleanup, and summary |
| `skills/rename-session/` | Programmatic session rename via `.jsonl` — called automatically by start/end-session |
| `skills/agent-teams/` | Patterns for orchestrating sub-agent teams |
| `skills/parallel-worktrees/` | Git worktree patterns for isolated parallel development |
| `skills/write-review-split/` | Parallel write + review workflow for quality |

## Environment Compatibility

| Environment | Shared CLAUDE.md | Shared Skills | Project CLAUDE.md | Project Skills |
|-------------|-----------------|---------------|-------------------|----------------|
| VS Code (home) | via `~/.claude/` + submodule | via `~/.claude/` + submodule | yes | yes |
| VS Code (work) | via submodule | via submodule | yes | yes |
| Claude Code web | via submodule | via submodule | yes | yes |

The submodule approach ensures preferences are available everywhere the repo is cloned, regardless of whether `~/.claude/` exists on that machine.
