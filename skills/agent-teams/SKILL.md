---
name: agent-teams
description: Orchestrate sub-agent teams for complex multi-part tasks — divide, conquer, synthesize
argument-hint: ""
user-invocable: true
---

# Agent Teams

Use this skill when a task is large enough to benefit from dividing work across multiple focused subagents. Each agent gets a clear scope, works independently, and reports back for synthesis.

## When to Use Agent Teams

- **Multi-component features**: frontend + backend + tests + docs
- **Research + implementation**: one agent researches best practices, another implements
- **Write + review**: one agent writes code, another reviews it (see `write-review-split` skill)
- **Parallel independent tasks**: linting, testing, building — all at once
- **Large refactors**: each agent handles a different module or layer

## When NOT to Use Agent Teams

- Single-file changes or small tasks — overhead isn't worth it
- Tightly coupled changes where agents would constantly conflict
- When the user wants to watch and guide each step interactively

## Team Patterns

### Pattern 1: Divide and Conquer

Best for tasks with clear, independent modules.

```
1. Plan: break task into N independent pieces
2. Spawn N subagents in parallel, each with:
   - Clear scope (what files/modules they own)
   - Clear deliverable (what "done" looks like)
   - Isolation if needed (worktree for code changes)
3. Collect results
4. Synthesize: resolve any conflicts, integrate pieces, verify cohesion
```

### Pattern 2: Pipeline

Best for sequential stages where each builds on the prior.

```
1. Agent A: Research / plan / scaffold
2. Agent B: Implement based on A's output
3. Agent C: Test and review B's implementation
4. Main agent: final synthesis and cleanup
```

### Pattern 3: Specialist Roles

Best for complex tasks requiring different expertise.

```
- Architect agent: designs the approach, identifies interfaces
- Builder agent(s): implement components
- Reviewer agent: checks for bugs, style, edge cases
- Doc agent: writes documentation based on final implementation
```

### Pattern 4: Parallel Explore

Best for evaluating multiple approaches.

```
1. Spawn 2-3 agents, each trying a different approach
2. Compare results
3. Pick the best approach, discard the rest
```

## Rules for Subagents

### Spawning
- Give each agent a **specific, scoped prompt** — not a vague "help with this"
- Include relevant context: file paths, constraints, style preferences
- Set `isolation: "worktree"` for agents making code changes to avoid conflicts
- Launch independent agents **in parallel** (multiple Agent calls in one message)

### Reporting
- Each agent should return a **concise summary**: what was done, what succeeded, what needs attention
- Don't dump entire file contents back — summarize and reference file paths

### Coordination
- The main (orchestrator) agent is responsible for:
  - Breaking down the task
  - Resolving conflicts between agent outputs
  - Merging worktree branches
  - Final quality check
- Subagents should NOT spawn their own subagents unless explicitly needed (avoid deep nesting)

### Error Handling
- If a subagent fails or gets stuck, the main agent should:
  - Assess the failure
  - Either retry with a refined prompt or handle the piece directly
  - Never silently drop a failed subtask

## Example: Full-Stack Feature

```
Task: "Add user profile page with avatar upload"

Main agent plan:
1. [Parallel] Spawn 3 agents:
   - Agent A (worktree): Backend API endpoints for profile CRUD + avatar upload
   - Agent B (worktree): Frontend React component for profile page
   - Agent C (no worktree): Research best avatar upload libraries, return recommendation
2. [Sequential] After all return:
   - Review Agent C's recommendation, apply to Agent A's branch
   - Merge Agent A's branch (backend)
   - Merge Agent B's branch (frontend), connecting to backend
3. [Parallel] Spawn 2 agents:
   - Agent D: Write integration tests
   - Agent E: Write documentation
4. Final review and commit
```
