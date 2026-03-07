---
name: executing-plans
description: Use when you have a written implementation plan (PBI with a Plan section) to execute. Covers picking up ready PBIs, executing tasks with TDD and batch checkpoints, error handling, and archiving completed work.
---

# Executing Plans — Codex Implementation

> Canonical skill: `skills/executing-plans/SKILL.md`

This implements `skills/executing-plans/SKILL.md` for Codex.

## Picking Up Ready PBIs

Before executing, check if the user pointed you at a specific plan or if you need to pick from the ready queue.

### Ready queue pickup

```bash
ls backlog/ready/pbi-*.md
```

Read the first file (alphabetical = lowest PBI number = highest priority). Extract the PBI ID from the header (`# PBI #NNN: ...`).

Move it to in_progress:

```bash
mv backlog/ready/pbi-NNN-*.md backlog/in_progress/
```

If an external tool is configured (see PO Coach adapters), also update its status to `in_progress` via the appropriate adapter.

Load the `## Plan` section from the PBI file for execution.

### Direct plan execution

If the user points you at a specific PBI file (e.g., "execute `backlog/ready/pbi-042-email-validation.md`"), use that file directly. Still move it to `backlog/in_progress/` before starting.

## Prerequisites

Before executing any plan:

1. **Read the PBI file.** Read the full PBI file. The plan is in the `## Plan` section.
2. **Set up the working environment.** If the plan specifies worktree execution, create a worktree before the first task:
   ```bash
   git worktree add worktrees/pbi-NNN-description -b pbi-NNN-description
   cd worktrees/pbi-NNN-description
   ```
   This gives you a clean branch based on HEAD, isolated from other work. All commits land on the worktree branch, not main.
3. **Verify the codebase.** Search for the file paths and patterns referenced in the plan to confirm they actually exist.

## Task Tracking

Maintain a live checklist in the PBI file's `## Plan` section. Update it as you complete each task:

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
Human: Execute backlog/in_progress/pbi-042-email-validation.md

Agent: [Read the PBI file, load the ## Plan section]
       [Explore codebase to verify plan accuracy]
       [Report any concerns or confirm ready to start]
```

Always verify before starting. Read files and search for patterns to check that the plan's assumptions about the codebase are still valid.

### 2. Execute Batches

Default batch size: 3 tasks. For each task:

1. **Read the task** from the plan
2. **Execute it** — edit files for modifications, create new files as needed, run commands in terminal
3. **Run the TDD cycle** — execute tests in terminal, confirm red then green
4. **Commit** — run `git add` and `git commit` in terminal with the plan's commit message
5. **Mark complete** — update the checklist in the PBI file

### 3. Report at Checkpoints

After each batch, present the report as described in `skills/executing-plans/SKILL.md`. Include:

- Actual test output (copy-paste from terminal output, not paraphrased)
- Commit messages and hashes
- Any deviations or concerns

Wait for the human to say "continue" before proceeding to the next batch.

### 4. Complete

When all tasks are done:

1. Run the full test suite in terminal
2. Verify every acceptance criterion — run commands, read files as needed
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

2. **File not found.** The plan references a path that does not exist. Search for files matching the expected name to find the actual location. Report the discrepancy and proposed correction. Wait for approval before proceeding.

3. **Merge conflict.** If main has moved since the plan was written and it affects the current task, report the conflict. Do not resolve merge conflicts silently.

4. **Build failure.** If a compile or build step fails, capture the full error output and report it. The plan may need adjustment.

## PBI Lifecycle Management

After all tasks complete and the completion report is presented:

1. **Update the backlog tool first** (if configured):
```bash
curl -X PATCH "$BACKLOG_API/api/backlog/$PBI_ID" \
  -H 'Content-Type: application/json' \
  -d '{"status": "done"}'
```

2. **Then archive the file:**
```bash
mv backlog/in_progress/pbi-NNN-*.md backlog/done/
git add backlog/ && git commit -m "backlog: complete pbi-NNN description"
```

If the backlog tool is unreachable, do **not** move the file. Add `## API Sync Needed` to the PBI file and tell the human partner.

## Example Session

```
Human: Build the next ready PBI

Agent: [Runs ls backlog/ready/pbi-*.md, finds pbi-042-email-validation.md]
       [Moves to backlog/in_progress/]
       [Reads PBI file, reviews 7 tasks in Plan section, verifies codebase]

       I've picked up PBI #42 (email validation) from the ready queue.
       7 tasks. File paths verified. Test framework confirmed (pytest).
       No concerns. Ready to start batch 1 (tasks 1-3)?
```
