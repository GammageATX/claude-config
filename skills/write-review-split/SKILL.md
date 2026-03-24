---
name: write-review-split
description: Split work into parallel write and review tracks for higher-quality output
argument-hint: ""
user-invocable: true
---

# Write-Review Split

Use this skill to run a parallel write + review workflow on significant code changes. One agent (or the main thread) writes the code, while a review agent catches issues concurrently.

## When to Use This

- Code changes touching **5+ files** or **100+ lines**
- Changes involving **complex logic**, **security-sensitive code**, or **API contracts**
- When you want a **second pair of eyes** without waiting for a human review cycle
- Refactors where subtle regressions are likely

## When NOT to Use This

- Small, straightforward changes (< 5 files, simple logic)
- Documentation-only changes
- Configuration changes

## The Pattern

### Step 1: Write Phase

The main agent (or a writer subagent) implements the change:

```
Writer agent:
- Implement the feature/fix
- Commit to a branch (use worktree if appropriate)
- Return: list of changed files, brief description of approach, any concerns
```

### Step 2: Review Phase (Parallel)

While the writer is working — or immediately after — spawn a review agent:

```
Review agent:
- Read all changed files (use git diff or read the files directly)
- Check for:
  * Bugs, logic errors, off-by-one mistakes
  * Missing error handling or edge cases
  * Security issues (injection, auth bypass, data exposure)
  * Performance concerns (N+1 queries, unnecessary allocations)
  * Style violations and naming inconsistencies
  * Missing tests for new behavior
  * Breaking changes to existing interfaces
- Return: list of issues found, severity (critical/warning/nit), suggested fixes
```

### Step 3: Synthesis

The main agent reviews the feedback and:
1. **Critical issues**: fix immediately before proceeding
2. **Warnings**: fix if straightforward, otherwise note for follow-up
3. **Nits**: batch and fix, or add to side quests for later

## Parallel Execution

The power of this pattern is running both tracks concurrently:

```
[Single message with two Agent calls]

Agent 1 (writer, worktree):
  "Implement the new caching layer for the API responses.
   Files to modify: src/api/cache.ts, src/api/middleware.ts, src/config.ts
   Requirements: ..."

Agent 2 (reviewer):
  "Review the changes being made to implement a caching layer.
   Once the writer agent returns, review all modified files.
   Focus on: cache invalidation correctness, memory leak potential,
   thread safety, error handling..."
```

If true parallel isn't possible (reviewer needs writer output), run sequentially but still as separate agents for clean separation of concerns.

## Review Checklist Template

The review agent should check:

- [ ] **Correctness**: does the code do what it's supposed to?
- [ ] **Edge cases**: empty inputs, null values, boundary conditions
- [ ] **Error handling**: are all failure modes covered?
- [ ] **Security**: no injection, no leaked secrets, proper auth checks
- [ ] **Performance**: no obvious N+1, no blocking calls in hot paths
- [ ] **Tests**: are new behaviors tested? Are existing tests still valid?
- [ ] **Interfaces**: do public APIs maintain backward compatibility?
- [ ] **Naming**: clear, consistent with project conventions
- [ ] **Documentation**: are complex decisions explained in comments?

## Rules

- **The reviewer should be constructive, not just critical.** Include suggested fixes, not just complaints.
- **Don't over-review trivial changes.** If the writer made a 3-line fix, a full review agent is overkill.
- **Track review findings.** If patterns of issues emerge (e.g., consistently missing error handling), propose a CLAUDE.md update to catch it earlier.
- **The user has final say.** Present review findings to the user for critical issues — don't silently fix architectural concerns.
