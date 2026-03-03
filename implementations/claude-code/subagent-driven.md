---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
---

# Subagent-Driven Development — Claude Code Implementation

This is the Claude Code implementation of the subagent-driven development skill. The canonical process definition is in `skills/subagent-driven/SKILL.md`. Read it first — this file adds Claude Code-specific execution details, not a separate process.

## How to Execute Each Role

### Orchestrator (you)

You are the orchestrator. You read the plan, dispatch workers via the **Agent** tool, dispatch reviewers via the **Agent** tool, and coordinate the sequence. You do not write implementation code directly.

### Worker Dispatch

Use the **Agent** tool to dispatch each worker. The worker prompt must be self-contained — the agent receives no prior context beyond what you provide.

Worker prompt template:

```
You are implementing a specific task as part of a larger plan. Follow these instructions exactly.

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
6. Self-review: re-read every file you changed, check for mistakes
7. Run the full test suite and verify all tests pass

REPORT:
When done, provide:
- Files created or modified (absolute paths)
- Tests written and their pass/fail status
- Full test suite output
- Any deviations from requirements (with justification)
```

### Spec Compliance Reviewer Dispatch

After the worker reports back, dispatch a spec compliance reviewer via the **Agent** tool.

Reviewer prompt template:

```
You are a spec compliance reviewer. Your job is to verify that an implementation matches its specification exactly — nothing more, nothing less.

ORIGINAL TASK SPEC:
[paste the exact requirements given to the worker]

WORKER REPORT:
[paste what the worker reported]

FILES TO REVIEW:
[list of files the worker changed]

REVIEW PROCESS:
1. Read every file the worker changed
2. For each requirement in the task spec, verify it is implemented correctly
3. Check that no unrequested changes were made
4. Verify test coverage matches the requirements
5. Check for any unrelated modifications

VERDICT:
Return one of:
- PASS — all requirements implemented, nothing extra, nothing missing
- FAIL — list each specific issue:
  - [requirement X]: [what is wrong]
  - [unrequested change]: [what was added that should not be there]
  - [missing]: [what was not implemented]
```

### Code Quality Reviewer Dispatch

After spec review passes, dispatch a code quality reviewer via the **Agent** tool.

Reviewer prompt template:

```
You are a code quality reviewer. Your job is to verify that the code is clean, correct, and well-tested. You are NOT checking whether the right thing was built — that has already been verified. You are checking whether it was built well.

FILES TO REVIEW:
[list of files the worker changed]

PROJECT CONVENTIONS:
[coding standards, patterns, relevant style from CLAUDE.md or project docs]

REVIEW CHECKLIST:
1. Code follows project conventions and existing patterns
2. Tests are meaningful — they test behavior, not just that code runs
3. No dead code, debug artifacts, commented-out code, or TODOs
4. Error handling is appropriate (not swallowed, not over-caught)
5. Architecture boundaries respected (no inappropriate cross-module dependencies)
6. Variable and function names are clear and consistent
7. No unnecessary complexity

VERDICT:
Return one of:
- PASS — code quality is acceptable
- FAIL — list each specific issue:
  - [file:line]: [what is wrong and how to fix it]
```

## Handling Review Failures

When a reviewer returns FAIL, re-dispatch the worker with the **Agent** tool:

```
You previously implemented [task title]. The [spec/quality] reviewer found issues that need fixing.

ISSUES TO FIX:
[paste the reviewer's specific issues]

ORIGINAL REQUIREMENTS (for reference):
[paste original requirements]

CONSTRAINTS:
- Fix ONLY the issues listed above
- Do NOT make unrelated changes
- Run the full test suite after fixes

REPORT:
- What you changed to address each issue
- Full test suite output
```

Then re-dispatch the same reviewer to verify the fixes.

## Step-by-Step Execution

1. **Read the plan** — load the implementation plan file, verify it has ordered tasks with acceptance criteria.

2. **Questions before dispatching** — For each task, check:
   - Is the task description unambiguous? (could a fresh agent interpret it two ways?)
   - Are dependencies between tasks explicit?
   - Does the worker prompt need project-specific conventions?
   - If any questions arise, ask the user before dispatching.

3. **For each task:**
   a. Dispatch worker via **Agent** tool with the worker prompt template
   b. Read the worker's report
   c. Dispatch spec compliance reviewer via **Agent** tool
   d. If spec review fails: re-dispatch worker with issues, re-review (max 3 cycles)
   e. Dispatch code quality reviewer via **Agent** tool
   f. If quality review fails: re-dispatch worker with issues, re-review (max 3 cycles)
   g. Log task as complete, advance to next

4. **After all tasks:** Run full test suite, compile summary, provide verification evidence to the user.

## What NOT to Do

- Do not write implementation code yourself — dispatch workers via **Agent**. You are the orchestrator.
- Do not skip spec review, even for "simple" tasks. The **Agent** tool call takes seconds. The working agreement is non-negotiable.
- Do not run quality review before spec review passes. Wrong order.
- Do not give workers vague prompts. Include file paths, patterns, constraints. The worker has no context beyond what you provide.
- Do not let more than 3 review-fix cycles pass without escalating to the user. Something is structurally wrong if a worker cannot satisfy a reviewer in 3 attempts.
- Do not use **Agent** for the brainstorming or design phases. Those are conversations with the user, not delegated tasks.
- Do not dispatch a worker if you have unresolved questions about the task. Ask the user first — a clarification round costs less than a wrong implementation + rework.
- Do not reverse the review order. Spec compliance FIRST, then code quality. Wrong order doubles the cost (see canonical skill for the waste example).
