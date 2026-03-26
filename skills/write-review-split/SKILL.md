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

## Graded Evaluation Rubric

Don't just use a pass/fail checklist. Grade each dimension on a 1-5 scale so the review produces actionable, weighted feedback. Weight criteria heavier in areas where the model tends to underperform (e.g., edge cases, originality, thorough testing).

| Dimension | Weight | 1 (Fail) | 3 (Acceptable) | 5 (Excellent) |
|-----------|--------|----------|-----------------|----------------|
| **Correctness** | High | Broken logic, wrong output | Works for happy path | Handles all paths correctly |
| **Edge Cases** | High | No consideration | Common cases handled | Boundary conditions, nulls, empty inputs all covered |
| **Security** | High | Injection/leak vulnerabilities | Basic auth/validation present | Defense in depth, no leaked secrets |
| **Testing** | Medium | No tests | Happy path tested | Edge cases tested, existing tests still valid |
| **Performance** | Medium | Obvious N+1 or blocking calls | No major issues | Optimized hot paths, efficient queries |
| **Interfaces** | Medium | Breaking changes unaddressed | Backward compatible | Clean API evolution with docs |
| **Craft** | Low | Copy-paste, inconsistent style | Follows conventions | Clean, idiomatic, well-named |
| **Documentation** | Low | No explanation of complex logic | Key decisions noted | Clear why-comments where needed |

**Scoring**: Flag any dimension scoring 1-2 as a **blocker**. Dimensions scoring 3 are **warnings**. Only pass the review if no blockers exist and the weighted average is ≥ 3.

## Output Interaction Requirement

The reviewer MUST interact with the output where possible — not just read the code:

- **Run the app/tests**: execute the code, hit endpoints, check the UI renders
- **Use available tools**: terminal, browser, MCP servers (Playwright, etc.) to verify behavior
- **Test like a user**: don't just verify the code compiles — verify it actually works as intended
- **If interaction isn't possible** (no dev server, no test harness), explicitly note this as a gap in the review and recommend what testing should happen before shipping

This mirrors Anthropic's finding that evaluators who can interact with the output catch significantly more issues than those who only read code.

## Anti-Sycophancy Instructions for Reviewer

Include these instructions in every reviewer agent prompt:

> **You are a skeptical QA reviewer, not a cheerleader.** Your job is to find problems.
> - Do NOT approve mediocre work. If something is bland, generic, or "AI slop," say so directly.
> - Do NOT talk yourself out of issues you've identified. If you notice a problem, it IS a problem — don't rationalize it away.
> - Do NOT praise the code to soften your critique. Lead with issues, then note what works.
> - If you find yourself writing "this is mostly fine but..." — stop. Investigate deeper.
> - Be specific: "the error handling on line 47 silently swallows the exception" not "error handling could be improved."

## Rules

- **The reviewer should be constructive, not just critical.** Include suggested fixes with severity ratings, not just complaints.
- **Don't over-review trivial changes.** If the writer made a 3-line fix, a full review agent is overkill.
- **Track review findings.** If patterns of issues emerge (e.g., consistently missing error handling), propose a CLAUDE.md update to catch it earlier.
- **The user has final say.** Present review findings to the user for critical issues — don't silently fix architectural concerns.
- **Weight toward known weaknesses.** If the model consistently struggles in certain areas (e.g., edge case handling, originality), weight those criteria heavier in the rubric.
