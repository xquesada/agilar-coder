---
name: subagent-driven-development
description: Use when executing implementation plans with multiple ordered tasks. The agent acts as orchestrator, worker, and reviewer in sequence for each task.
---

<!-- Adapted from Claude Code: subagent dispatch replaced with sequential self-execution. The orchestrator/worker/reviewer roles are performed by the same agent in sequence. -->

# Subagent-Driven Development — Codex Implementation

> Canonical skill: `skills/subagent-driven/SKILL.md`
>
> This file adapts the subagent-driven pattern for Codex, which does not support dispatching sub-agents. Instead, the agent performs all three roles (orchestrator, worker, reviewer) sequentially within a single session.

## How It Works in Codex

In Claude Code, the orchestrator dispatches separate agents for worker and reviewer roles. In Codex, you perform all three roles yourself, switching hats explicitly:

1. **Orchestrator** — read the plan, decide task order, manage the queue
2. **Worker** — implement the task (write tests, write code, run tests)
3. **Reviewer** — review your own work against the spec, then for code quality

The key discipline is **role separation**: when reviewing, re-read every file you changed as if seeing it for the first time. Do not assume correctness from memory.

## Worker Self-Briefing Checklist

Before implementing each task, brief yourself using this template:

```
TASK: [task title]

CONTEXT:
- Project root: [absolute path]
- [relevant file paths and their purpose]
- [relevant patterns/conventions to follow]
- [dependencies on previous tasks, if any]

REQUIREMENTS:
1. [specific requirement]
2. [specific requirement]
3. [specific requirement]

CONSTRAINTS:
- Do NOT modify files outside this task's scope
- Do NOT add features not listed in requirements
- Follow the existing patterns in [relevant files]

PROCESS:
1. Read the relevant source files to understand current state
2. Write a failing test for the first requirement
3. Implement until the test passes
4. Refactor if needed
5. Repeat for each requirement
6. Run the full test suite and verify all tests pass
```

## Spec Compliance Review

After completing implementation, switch to reviewer mode. Re-read every file you changed.

Review checklist:

```
ORIGINAL TASK SPEC:
[the exact requirements from the self-briefing]

REVIEW PROCESS:
1. Read every file you changed — do not rely on memory
2. For each requirement in the task spec, verify it is implemented correctly
3. Check that no unrequested changes were made
4. Verify test coverage matches the requirements
5. Check for any unrelated modifications

VERDICT:
- PASS — all requirements implemented, nothing extra, nothing missing
- FAIL — list each specific issue:
  - [requirement X]: [what is wrong]
  - [unrequested change]: [what was added that should not be there]
  - [missing]: [what was not implemented]
```

If the verdict is FAIL, fix the issues and re-review (max 3 cycles).

## Code Quality Review

After spec review passes, switch to quality reviewer mode.

Review checklist:

```
REVIEW CHECKLIST:
1. Code follows project conventions and existing patterns
2. Tests are meaningful — they test behavior, not just that code runs
3. No dead code, debug artifacts, commented-out code, or TODOs
4. Error handling is appropriate (not swallowed, not over-caught)
5. Architecture boundaries respected (no inappropriate cross-module dependencies)
6. Variable and function names are clear and consistent
7. No unnecessary complexity

VERDICT:
- PASS — code quality is acceptable
- FAIL — list each specific issue:
  - [file:line]: [what is wrong and how to fix it]
```

If the verdict is FAIL, fix the issues and re-review (max 3 cycles).

## Step-by-Step Execution

1. **Read the plan** — load the implementation plan file, verify it has ordered tasks with acceptance criteria.

2. **Questions before starting** — For each task, check:
   - Is the task description unambiguous?
   - Are dependencies between tasks explicit?
   - Does the task need project-specific conventions?
   - If any questions arise, ask the user before starting.

3. **For each task:**
   a. Brief yourself using the worker checklist
   b. Implement the task (worker role)
   c. Run spec compliance review (reviewer role)
   d. If spec review fails: fix issues, re-review (max 3 cycles)
   e. Run code quality review (reviewer role)
   f. If quality review fails: fix issues, re-review (max 3 cycles)
   g. Log task as complete, advance to next

4. **After all tasks:** Run full test suite, compile summary, provide verification evidence to the user.

## Batch Ready Queue Execution

When told to process the ready queue ("build all ready PBIs", "work on ready PBIs"):

### 1. List ready PBIs

```bash
ls backlog/ready/pbi-*.md
```

### 2. For each PBI file

```
Read file, extract PBI number from # PBI #NNN: header
mv backlog/ready/pbi-NNN-*.md backlog/in_progress/
If external tool: sync status to in_progress
```

### 3. Implement

Use the `## Plan` section from the PBI file as the task list. Include the PBI's acceptance criteria. Execute each task through the worker → spec review → quality review cycle.

### 4. After completion

```
mv backlog/in_progress/pbi-NNN-*.md backlog/done/
git add backlog/ && git commit -m "backlog: complete pbi-NNN description"
If external tool: sync status to done
```

## What NOT to Do

- Do not skip spec review, even for "simple" tasks. The working agreement is non-negotiable.
- Do not run quality review before spec review passes. Wrong order.
- Do not rely on memory during review — re-read every changed file.
- Do not let more than 3 review-fix cycles pass without escalating to the user. Something is structurally wrong if the implementation cannot satisfy review in 3 attempts.
- Do not skip the self-briefing checklist. It prevents scope drift.
- Do not reverse the review order. Spec compliance FIRST, then code quality. Wrong order doubles the cost (see canonical skill for the waste example).
