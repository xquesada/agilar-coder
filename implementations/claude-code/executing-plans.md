---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans — Claude Code Implementation

This implements `skills/executing-plans/SKILL.md` for Claude Code.

## Prerequisites

Before executing any plan:

1. **Read the plan file.** Use Read to load the full plan document.
2. **Set up the working environment.** If the plan specifies worktree execution, create a worktree before the first task:
   - Use EnterWorktree to create an isolated worktree
   - This gives you a clean branch based on HEAD, isolated from other work
   - All commits land on the worktree branch, not main
3. **Verify the codebase.** Use Grep and Glob to confirm the file paths and patterns referenced in the plan actually exist.

## Task Tracking

Use TodoWrite to maintain a live checklist of plan tasks. Update it as you complete each task:

```
- [x] Task 1: Create user schema type
- [x] Task 2: Write validation tests (red)
- [x] Task 3: Implement validation (green)
- [ ] Task 4: Write API handler tests (red)
- [ ] Task 5: Implement API handler (green)
```

This gives the human partner visibility into progress without needing to ask.

## Execution Flow

### 1. Load and Review

```
Human: Execute the plan at docs/plans/2026-02-28-pbi-042-email-validation.md

Agent: [Read the plan file]
       [Explore codebase to verify plan accuracy]
       [Report any concerns or confirm ready to start]
```

Always verify before starting. Use Read, Grep, and Glob to check that the plan's assumptions about the codebase are still valid.

### 2. Execute Batches

Default batch size: 3 tasks. For each task:

1. **Read the task** from the plan
2. **Execute it** — use Edit for modifications, Write for new files, Bash for commands
3. **Run the TDD cycle** — Bash to run tests, confirm red then green
4. **Commit** — Bash to git add and git commit with the plan's commit message
5. **Mark complete** — update TodoWrite

### 3. Report at Checkpoints

After each batch, present the report as described in `skills/executing-plans/SKILL.md`. Include:

- Actual test output (copy-paste from Bash output, not paraphrased)
- Commit messages and hashes
- Any deviations or concerns

Wait for the human to say "continue" before proceeding to the next batch.

### 4. Complete

When all tasks are done:

1. Run the full test suite via Bash
2. Verify every acceptance criterion — use Bash, Read, or browser tools as needed
3. Present the completion report with evidence

## Branching and Merging

### Worktree Execution (default for multi-step plans)

The plan executes on a worktree branch. When complete and approved:

1. The human reviews the code (see `skills/code-review/`)
2. If approved, merge the worktree branch to main
3. Follow the finishing-a-development-branch process: rebase if needed, merge, verify main is green, clean up the worktree

### Trunk Execution (solo mode, small plans)

If the plan specifies trunk execution or the human confirms working on main directly:

- Commit directly to main after each green task
- No merge step needed
- Higher risk — every commit is immediately on the trunk

## Error Handling

When something goes wrong during execution:

1. **Test failure.** Run the failing test in isolation. Read the error output carefully. Check if the plan's code matches the actual codebase state. Report findings to the human — do not debug for more than 5 minutes without reporting.

2. **File not found.** The plan references a path that does not exist. Use Glob to find the actual location. Report the discrepancy and proposed correction. Wait for approval before proceeding.

3. **Merge conflict.** If main has moved since the plan was written and it affects the current task, report the conflict. Do not resolve merge conflicts silently.

4. **Build failure.** If a compile or build step fails, capture the full error output with Bash and report it. The plan may need adjustment.

## Subagent Execution

For plans with independent task groups, the orchestrator session can spawn subagents:

1. **Identify independent groups.** Tasks that do not share files or depend on each other can run in parallel.
2. **Create a worktree per subagent.** Each subagent works in isolation.
3. **Spawn via TaskCreate.** Pass the plan file path and the specific task numbers to execute.
4. **Merge results.** The orchestrator merges each worktree branch to main, running the full test suite after each merge.

## Example Session

```
Human: Execute docs/plans/2026-02-28-pbi-042-email-validation.md

Agent: [Reads plan, reviews 7 tasks, verifies codebase]

       I've reviewed the plan. 7 tasks for PBI #42 (email validation).
       File paths verified. Test framework confirmed (pytest).
       No concerns. Ready to start batch 1 (tasks 1-3)?