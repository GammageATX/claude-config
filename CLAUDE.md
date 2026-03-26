# User Preferences — Michael Gammage

## Environment

- I primarily use the **VS Code extension** for Claude Code, not the terminal directly.
- Because of this, I cannot always see inline terminal output. Use **TodoWrite** for status tracking on all multi-step tasks — it renders as a visible widget in VS Code.
- Always echo a brief status line to the terminal after completing each major step (e.g., `echo "✓ Step 3/5: Tests passing"`), as a fallback if the todo widget isn't visible.

## Workflow Rules

### Status Updates & Progress

- **Loop status updates automatically on larger tasks.** For any task with 4+ steps, provide a status update after each step — don't wait until the end to report.
- **Use TodoWrite liberally.** If in doubt, create a todo list. I'd rather see too many status updates than wonder what's happening.

### Parallel Execution

- **Background non-sequential steps.** When steps are independent (e.g., running tests while drafting docs, linting while committing), use `run_in_background` or launch parallel subagents. Don't serialize work that can be parallelized.
- **Use worktrees for major improvements.** When making large or risky changes, prefer `--worktree` isolation so the main branch stays clean. See the `parallel-worktrees` skill for the full pattern.
- **Split write and review when appropriate.** On significant code changes, consider spawning a parallel review subagent to catch issues while the main agent continues working. See the `write-review-split` skill.

### Sub-Agents & Teams

- **Establish sub-agent teams when appropriate.** For complex multi-part tasks, spin up focused subagents rather than doing everything sequentially in one context. Each subagent should have a clear, scoped responsibility.
- **Subagents should report back concisely** — a summary of what was done, what succeeded, and what needs attention.

### Context Hygiene

- **Suggest starting fresh after a few big changes.** If the conversation has accumulated significant context from multiple large tasks, proactively suggest `/compact` or starting a new session. Don't wait for me to notice degradation.
- **Watch for context window pressure.** If you notice you're approaching limits or responses are getting less precise, say so and recommend a fresh start.
- **Monitor for context anxiety.** As the context window fills, models change behavior in predictable ways: rushing through steps, giving shorter/terser responses, declaring things "done" prematurely, skipping planned work, or wrapping up the conversation early. If you notice any of these symptoms in yourself, **flag it immediately** — don't let quality degrade silently. Use `/compact` if on-track (preserves summary), `/clear` if off-track (clean slate), or suggest a fresh session for a full context reset.
- **`/clear` vs `/compact` guidance.** Use `/clear` when the conversation has gone off-track and you need a clean slate. Use `/compact` when on-track but the context is getting large — it summarizes and preserves continuity. Default to `/compact` unless the thread has drifted significantly.

### Learning from Mistakes

- **Update CLAUDE.md when I call out mistakes.** If I correct you on a preference, pattern, or approach, propose adding it to this file (or the project-level CLAUDE.md) so the same mistake doesn't happen again. Frame it as: "Want me to add a note about this to CLAUDE.md so I remember next time?"
- **Update project CLAUDE.md with discovered patterns.** If we discover something important about a codebase during a session (build quirks, naming conventions, gotchas), offer to persist it.

### Planning & Execution

- **Default to Plan Mode for multi-file changes.** Before touching 3+ files, outline the plan first. I want to approve the approach before execution begins.
- **Use ultrathink for complex reasoning.** On architecture decisions, debugging sessions, or anything requiring deep analysis, use extended thinking.
- **Define "Done" before executing.** For any non-trivial task, write 3-5 concrete acceptance criteria before starting work. This prevents goalpost-moving mid-session where work gets declared "done" when it's only partially complete. Check work against these criteria before moving on.
- **Expand scope for ambitious tasks.** If a task is a full feature or multi-sprint goal, don't jump in from a one-liner. Spend time expanding it into a structured spec (components, interfaces, data flow, dependencies) before writing code. A one-sentence prompt dramatically underscopes the work.

### Self-Evaluation & Quality

- **Don't self-approve mediocre work.** Agents tend to praise their own output, even when quality is obviously lacking. Be honest — if something is bland, incomplete, or "AI slop," say so. Don't talk yourself out of issues you've identified.
- **Interact with output, don't just read code.** Where possible, actually run what was built — execute tests, hit endpoints, check the UI. Reading code and assuming it works is insufficient.
- **Use adversarial review for significant work.** For large features or complex changes, use the `write-review-split` skill with a dedicated skeptical reviewer. The tension between builder and reviewer improves quality — just like GAN networks.
- **Grade work on concrete criteria, not vibes.** Instead of "is this good?", evaluate against specific dimensions (correctness, edge cases, security, testing, craft). See the graded rubric in the `write-review-split` skill.

### Harness Evolution

- **Every workaround encodes a model assumption.** The patterns in these skills (context resets, multi-agent review, contract negotiation) exist because models have limitations. As models improve, some of these patterns become unnecessary overhead. Periodically revisit whether a pattern is still earning its keep.
- **Match harness complexity to task difficulty.** Don't over-engineer the orchestration for simple tasks. A solo agent can handle straightforward work. Reserve multi-agent patterns (planner → builder → evaluator) for tasks that genuinely stretch the model's capabilities.

## Session Management

- I use custom `/start-session` and `/end-session` skills for session lifecycle. Respect the quest system (main quest / side quest) defined in those skills.
- Session context files: `NEXT_SESSION.md`, `TODO.md`, `SIDE_QUESTS.md` — always check for these at session start.

### Conversation Naming

- **Your very first output line in a new session MUST be a descriptive summary** — NOT a generic phrase like "Starting new session" or "Initializing session." The Claude desktop app auto-titles conversations based on early output, so the first line determines the conversation name.
- Derive the title from `NEXT_SESSION.md` content: combine the project name (or folder name) with the top priority or focus area.
- Format: `<Project Name>: <Focus Area>` — e.g., "Golf Shot Tracker: Sprint 5 Practice Plan Generator" or "PDB Agents: Pipeline refactoring and test fixes"
- Print this descriptive line BEFORE the session banner.
- **After the user confirms their focus**, use the `rename-session` skill to programmatically set the session title. This is critical for VS Code (where the first-line trick doesn't work) and also reinforces the title in terminal sessions. The rename happens after focus confirmation so the title reflects the actual chosen work, not just the initial suggestion.

## Build Philosophy

- **Sprint-based big vision.** I like to define an ambitious end-state, break it into sprints, and execute sprint-by-sprint. Each session should deliver something real and working — not just plans or stubs.
- **Show me something I can react to.** I need to see semi-working output to critique and give polishing thoughts. Don't wait for perfection — get it functional, then I'll steer refinements.
- **Capture ideas as they come.** I generate a lot of ideas mid-session that aren't the current focus. Pencil them into `SIDE_QUESTS.md` or `TODO.md` backlog immediately so nothing gets lost, then stay on task.
- **Tokens are paid for — use them.** When I'm vibe coding, don't be conservative. Go big on planning, generate comprehensive code, use subagents for parallel work. I'd rather over-build and trim than under-build and iterate slowly.
- **Max plan = max ambition.** When I say I want a "max plan" or "big vision," that means: lay out the full end-state, design all the sprints, and start executing. Don't sandbag scope — I'll tell you if it's too much.

## Frontend Style Preferences

- **Rich & polished.** I want visual depth — gradients, subtle animations, shadows, and a production feel even in prototypes. Not flat/minimal.
- **Dark mode by default.** Always start with a dark theme. Light mode is secondary or optional.
- **Type safety everywhere.** TypeScript over JavaScript, always. Strict mode, proper interfaces, no `any` unless absolutely necessary.
- **Component library:** prefer shadcn/ui + Tailwind as the base, but customize beyond the defaults — I want things to look unique, not like every other shadcn app.
- **Responsive but desktop-first** for my projects (golf tracker is used on an outdoor TV, agents project is a desktop admin portal).

## Code Style Preferences

- Clear over clever. Readable code with good names over terse one-liners.
- Comments should explain *why*, not *what*.
- If I don't specify a language/framework, ask before assuming.
- **Type safety:** TypeScript for frontend, typed Python (type hints + Pydantic models) for backend. Strict schemas for API contracts.
- **Docs alongside code.** Generate inline docs, docstrings, README sections, and API docs as you build — not as a separate "documentation pass" later.

## Communication Style

- Be direct. Skip the preamble on technical responses.
- When presenting options, lead with your recommendation and explain why.
- Don't ask for permission on small decisions — just do them and mention what you did.
- DO ask for permission on architectural decisions, file deletions, or anything irreversible.
