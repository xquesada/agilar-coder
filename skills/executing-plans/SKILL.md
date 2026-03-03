# Executing Plans

Load an implementation plan, review it critically, execute tasks in batches, and report back for human review at checkpoints.

## Overview

This skill is the counterpart to `skills/sprint-planning/`. A plan defines WHAT to build. This skill defines HOW to execute it reliably. The executor — human, agent, or subagent — follows the plan step by step, never deviating without explicit consent.

Plans implement PBIs (see `SCRUM.md`). Completing a plan means delivering a PBI toward its Definition of Done. The executor is responsible for the mechanical execution; the human partner remains responsible for design decisions and final acceptance.

## Picking Up Ready PBIs

The ready queue is `backlog/ready/`. It contains PBI files with approved plans, waiting for execution.

### "Build next ready PBI" flow

1. Scan `backlog/ready/` for PBI files (`pbi-*.md`)
2. Pick the first file in alphabetical order (lowest PBI number = highest priority)
3. Move it to `backlog/in_progress/`
4. If an external tool is configured, update its status to `in_progress`
5. Proceed to Step 1 (load and review the plan from the PBI file)

### "Build all ready PBIs" flow

1. List all files in `backlog/ready/`
2. Process sequentially (solo) or dispatch to parallel agents (multi-agent). Each PBI goes through the full skill.
3. Priority: process in filename order (lowest PBI number first)

### Priority

Always process PBIs in filename order. `pbi-042-...` before `pbi-043-...`. This ensures the PO's priority order is respected.

## Step 1: Load and Review Plan

Before writing a single line of code, read the entire plan and evaluate it critically. The plan is the `## Plan` section of the PBI file. Acceptance criteria are in the same file.

### What to Check

1. **Goal clarity.** Do you understand what the codebase should look like when the plan is complete?
2. **Task completeness.** Does every acceptance criterion map to at least one task?
3. **Task ordering.** Are dependencies correct? Will task N work if tasks 1 through N-1 are complete?
4. **Verification steps.** Does every task have a way to verify it worked?
5. **Codebase alignment.** Do the file paths, patterns, and conventions in the plan match the actual codebase?

### If You Find Problems

Raise them BEFORE starting execution. List every concern:

```
I reviewed the plan and have the following concerns before starting:

1. Task 4 references src/handlers/user.go but the project uses src/handler/ (singular, no subdirectory)
2. Task 7 has no verification step
3. The plan does not cover acceptance criterion #3 (error handling for duplicate emails)

Should I proceed with adjustments, or would you like to revise the plan?
```

Never silently fix plan problems during execution. The human partner needs to know the plan was imperfect so they can improve future plans.

## Step 2: Execute a Batch

Execute tasks in batches. The default batch size is 3 tasks. The human partner can adjust this.

### For Each Task in the Batch

1. **Read the task.** Understand what it asks.
2. **Follow it exactly.** Do not add features, refactor unrelated code, or "improve" the plan. Do what it says.
3. **TDD cycle.** If the task includes tests (it should — see `skills/tdd/`):
   - Write the test first
   - Run it — confirm it FAILS
   - Write the minimal production code
   - Run it — confirm it PASSES
4. **Verify.** Run the verification step described in the task. Capture the output.
5. **Commit.** Use the commit message from the plan. Commit after each green task — not at the end of the batch.

### What "Follow Exactly" Means

- Use the exact file paths in the plan
- Write the code specified (or following the spec) in the plan
- Run the exact verification commands in the plan
- Do not rename things, reorganize imports, or "clean up" unless the task says to

If you discover something the plan got wrong (a file path that does not exist, a function signature that changed), STOP and report it. Do not improvise a fix.

## Step 3: Report

After completing a batch, report to the human partner:

```
Batch complete: Tasks [N] through [M]

Implemented:
- Task N: [one-line summary of what was done]
- Task N+1: [one-line summary]
- Task M: [one-line summary]

Verification:
- [paste actual test output or verification evidence]

All tests passing: yes/no
Commits: [list commit hashes or messages]

Ready for next batch, or would you like to review first?
```

The report includes ACTUAL verification output — not "tests should pass" but the real output from running them. This satisfies the verification working agreement (see `skills/verification/`).

## Step 4: Continue Based on Feedback

The human partner reviews the report and decides:

- **"Continue"** — execute the next batch
- **"Let me review the code first"** — pause and wait for feedback
- **"Adjust the plan"** — the human modifies remaining tasks before continuing
- **"Stop"** — halt execution. Remaining tasks are not done. The PBI stays in progress.

Never continue to the next batch without the human partner's go-ahead. The review checkpoint is not optional — it is the mechanism by which the human maintains design authority.

## Step 5: Complete Development

When all tasks are done:

1. **Run the full test suite.** Not just the tests from the plan — ALL tests. Capture the output.
2. **Verify all acceptance criteria.** Go through each criterion from the PBI and confirm it is met. Provide evidence for each one.
3. **Report completion.**

```
Plan complete.

All tasks executed: [N] tasks
All tests passing: [paste full test suite output]

Acceptance criteria verification:
- AC1: [criterion] — VERIFIED: [evidence]
- AC2: [criterion] — VERIFIED: [evidence]
- AC3: [criterion] — VERIFIED: [evidence]

The PBI is ready for code review (skills/code-review/) and final acceptance.
```

The PBI is not done yet. The Definition of Done (`SCRUM.md`) requires code review and human acceptance. Completing the plan means the implementation is ready for review — not that the PBI is closed.

## PBI Lifecycle Management

The executing-plans skill manages PBI file movement as part of execution — this is lifecycle management, not part of the Definition of Done.

### On pickup

Move the PBI file from `backlog/ready/` to `backlog/in_progress/`. If an external tool is configured, sync the status to `in_progress` via the appropriate adapter (see `skills/po-coach/`).

### On completion

Move the PBI file from `backlog/in_progress/` to `backlog/done/`. If an external tool is configured, sync the status to `done`.

The `backlog/done/` folder serves as an execution archive. The PBI still needs code review and human acceptance before it meets the Definition of Done — but the implementation work is complete and the file is archived.

## When to Stop and Ask for Help

Stop execution and ask the human partner when:

- **A test fails unexpectedly.** If the plan says a test should pass and it does not, something is wrong. Do not debug endlessly — report the failure.
- **A file path does not exist.** The plan references something that is not in the codebase. Report it.
- **A task is ambiguous.** If you cannot determine exactly what to do, ask. Do not guess.
- **The codebase changed.** If another developer or agent modified files the plan depends on, the plan may be stale. Report the conflict.
- **You hit a dead end.** After a reasonable effort (investigate the issue, read relevant code, check error messages), if you cannot make progress, report what you found and ask for guidance.

The human partner would rather be interrupted than deal with the consequences of a wrong guess.

## When to Revisit Earlier Steps

Sometimes a later task reveals that an earlier task was wrong or incomplete. When this happens:

1. **Stop the current task.**
2. **Identify the problem.** Which earlier task is affected? What needs to change?
3. **Report to the human partner.** Explain the issue and propose a fix.
4. **Wait for approval.** The human decides whether to go back and fix, adjust the plan, or proceed differently.

Do not silently patch earlier work. The human partner needs to understand the deviation from the plan.

## Execution Rules

1. **Never start on the main branch without consent.** If the plan specifies worktree execution or if you are in a multi-agent setup, verify the branching strategy before the first commit.
2. **Commit after every green task.** Not at the end of the batch. Not at the end of the plan. After every task where tests pass.
3. **Do not scope-creep.** Execute the plan. Only the plan. If you see an improvement opportunity, note it for the human — do not implement it.
4. **Preserve the plan.** Do not modify the plan document during execution. If adjustments are needed, the human partner makes them.
5. **One task at a time.** Even within a batch, execute sequentially. Finish task N before starting task N+1. Context-switching within a batch defeats the purpose of structured execution.

## Anti-Patterns

- **Skipping verification.** Every task has a verification step. Run it. Capture the output. Do not skip it because "it obviously works."
- **Batch-committing.** Committing at the end of a batch instead of after each task. If something breaks, you cannot isolate which task caused it.
- **Silent deviation.** Changing the approach without telling the human. Even if your way is better, the human needs to know.
- **Continuing past failure.** A test fails and you move on to the next task anyway. Stop. Report. Wait.
- **Gold-plating.** Adding error handling, logging, or features not in the plan. Note the improvement for later. Execute the plan as written.
