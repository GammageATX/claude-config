# AI Agent Stack Overview (March 2026)

Four open-source projects shaping how developers build with AI agents — from solo dev productivity to fully autonomous companies.

---

## 1. GStack — YC Office Hours for Solo Devs

**Author:** Gary Tan (Y Combinator President)
**GitHub:** ~50,000 stars (within weeks of launch)
**Category:** Claude Code process layer / startup methodology

### What It Is

GStack is a process framework installed on top of Claude Code (or Codex, etc.) that gives solo developers the power of an entire team. It encodes Gary Tan's experience overseeing thousands of YC startups (Airbnb, DoorDash, etc.) into structured prompts and role-based workflows.

### Key Concept: Process, Not Tools

GStack is explicitly described as a **process**, not a collection of individual tools. The workflow is designed to fine-tune your idea and market opportunity *before* writing code.

### Roles & Workflows

Each role is a dedicated prompt/markdown file that gives the AI agent specific direction through a particular lens:

| Role | Purpose |
|------|---------|
| **Office Hours** | 6 forcing questions that reframe your product before writing code (modeled after YC partner sessions) |
| **Plan CEO Review** | Rethink the problem from a CEO perspective |
| **10-Star Product** | Find the "10-star experience" (inspired by Airbnb's Brian Chesky) |
| **Engineering Manager** | Engineering leadership perspective |
| **Senior Designer** | Design-focused review |
| **Design Partner** | External design partner feedback simulation |
| **Staff Engineer** | Deep technical review |
| **Debugger** | Focused debugging assistance |

### Installation

Copy-paste a prompt into Claude Code. It asks a few questions, grants permissions, and installs. Then use `/gstack` subcommands (e.g., `/off-hours`).

### Why It Matters

- Prompt engineering at its finest — role-based prompts that shape how the agent views your codebase
- Open source = you can read how Gary Tan thinks about startups
- Rapid iteration — updates seemingly every day
- Accepting contributors

---

## 2. Hermes Agent — Self-Improving Personal AI

**Author:** Nous Research
**GitHub:** ~12,000 stars (days after launch)
**Category:** Personal AI agent framework (OpenClaw alternative)

### What It Is

Hermes Agent is a full agent framework/operating system — not just a coding assistant. It's comparable to OpenClaw but with a standout feature: a built-in self-improving learning loop.

### Key Features

**Terminal Interface (TUI):**
- Multi-line editing, command autocomplete
- Conversation history
- Interrupt and redirect
- Streaming tool output

**Multi-Platform Chat Gateway:**
- Telegram, Discord, Slack, WhatsApp, Signal, CLI
- Single gateway for all platforms (similar to OpenClaw)

**Self-Improving Learning Loop (the differentiator):**
- Agent-curated memory with periodic nudges
- Autonomous skill creation after complex tasks
- Skills self-improve during use
- Searches its own past conversations
- Builds a deepening model of who you are across sessions

**Other Capabilities:**
- Scheduler / cron jobs
- Parallel sub-agent delegation
- Run anywhere (local, cloud, hybrid)
- OpenClaw migration path (import existing workflows and memories)

### Why It Matters

- The learning loop is the standout — the agent gets better the more you use it
- Migration from OpenClaw lowers the switching cost
- Raises the question: the industry needs a **standard for agent portability** so you can switch frameworks without starting from scratch

---

## 3. Superpowers — TDD-First Claude Code Enhancement

**Author:** Ora
**GitHub:** ~115,000 stars (the most popular of the four)
**Category:** Claude Code plugin for structured development workflows

### What It Is

Superpowers is a Claude Code plugin that enforces structured, test-driven development workflows. It emphasizes TDD (Test-Driven Development), YAGNI (You Aren't Gonna Need It), and DRY principles.

### The Self-Aware Pitch

> "Your agent puts together an implementation plan that's clear enough for an enthusiastic junior engineer with poor taste, no judgment, no project context, and an aversion to testing to follow."

### Workflow

1. **Brainstorm** — Activates before writing code, refines rough ideas, explores alternatives, presents design in sections for validation, saves a design document
2. **Work Trees** — Uses git worktrees by default for parallelization
3. **Plan** — Writes structured implementation plans
4. **Execute** — Implements with TDD discipline
5. **Code Review** — Automated review step
6. **Finish** — Completes the branch

### Installation

```bash
claude plugin install superpowers@claude-plugins-official
```

Then use `/s superpowers brainstorm` (or other subcommands) to access the workflow.

### Why It Matters

- Enforces development best practices that agents tend to skip
- Worktree-first approach aligns with parallel agent execution
- Plugin format makes installation trivial
- The brainstorm step prevents the common failure mode of agents jumping straight into code

---

## 4. Paperclip — Zero-Human Company Orchestration

**GitHub:** ~33,000 stars
**Category:** Multi-agent company orchestration

### What It Is

Paperclip is a Node.js server + React UI that orchestrates a team of AI agents to run an entire business. If OpenClaw is an employee, Paperclip is the company.

### Architecture

- **Org Chart:** CEO, CMO, CTO, Engineers — all AI agents
- **Mixed Models:** Different agents can use different models (Claude, Codex, etc.)
- **Ticketing System:** Issues are created, assigned to agent teams, built, and deployed
- **Goal-Driven:** All agents reference your high-level company goals
- **Atomic Work:** All work units are atomic and trackable
- **Token Tracking:** Dashboard shows spend per agent (this gets expensive)
- **Projects:** Multiple projects with dedicated agent teams

### Dashboard Features

- Issues and engineering queue
- Org chart visualization
- Project management
- Cost tracking per agent
- Goal alignment tracking

### Getting Started

```bash
paperclip-ai onboard -y
```

### Upcoming Roadmap

- Plugin system (knowledge base, custom tracing, queues)
- OpenClaw-style agents for more capability
- Direct chat with agents (Telegram, etc.)
- `companies.sh` — export/import entire organizations
- Skills manager
- Scheduling routines
- Better budgeting

### Reality Check

This is experimental. The likelihood of "wake up and find money in your bank account" is low without significant effort. These zero-human company projects (including the closed-source Pulsia by Ben Sarah) still have rough edges. But as a vision of where multi-agent orchestration is heading, it's compelling.

---

## Comparison Matrix

| | GStack | Hermes Agent | Superpowers | Paperclip |
|---|--------|-------------|-------------|-----------|
| **Stars** | ~50k | ~12k | ~115k | ~33k |
| **Focus** | Startup process | Personal AI OS | Dev workflow | Company orchestration |
| **Installs on** | Claude Code | Standalone | Claude Code (plugin) | Standalone (Node.js) |
| **Key Innovation** | YC methodology as prompts | Self-improving loop | TDD-first workflow | Multi-agent org chart |
| **Best For** | Solo founders | Power users wanting a personal AI | Developers wanting structured TDD | Experimenting with autonomous teams |
| **Complexity** | Low | Medium | Low | High |

---

## Relevance to claude-config

Several patterns from these projects align with or extend patterns already in this repo:

- **GStack's role-based prompts** — Similar to the `skills/` approach here, but applied to business roles rather than dev workflows. Could inspire new skills like `office-hours/` or `product-review/`.
- **Superpowers' brainstorm-first workflow** — Aligns with the "Default to Plan Mode for multi-file changes" preference in CLAUDE.md. The brainstorm step could be adapted into a skill.
- **Superpowers' worktree-first approach** — Directly maps to `skills/parallel-worktrees/`.
- **Hermes Agent's learning loop** — The self-improving memory pattern is something to watch. Currently, claude-config handles cross-session context via `NEXT_SESSION.md` and `CLAUDE.md` updates, but a more automated approach could be valuable.
- **Paperclip's sub-agent org chart** — Extends the `skills/agent-teams/` pattern to a full organizational model with budgeting and goal alignment.
