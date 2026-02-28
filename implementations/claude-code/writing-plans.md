---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans — Claude Code Implementation

This implements `skills/writing-plans/SKILL.md` for Claude Code.

## Entering Plan Mode

When the human asks you to plan (or when a PBI is complex enough to warrant a plan), switch into plan mode:

1. **Read the PBI.** Understand the goal, acceptance criteria, and any design decisions already made.
2. **Explore the codebase.** Use Read, Grep, and Glob to understand existing patterns, conventions, file structure, and test infrastructure. The plan must fit the codebase as it is, not as you imagine it.
3. **Draft the plan.** Follow the structure from `skills/writing-plans/SKILL.md` — header with Goal/Architecture/Tech Stack/Acceptance Criteria, then numbered tasks at 2-5 minute granularity.
4. **Save the plan to a file.** Write it to `docs/plans/` or a location the human specifies. Plans are artifacts — they persist across sessions.

## Plan File Convention

Save plans as markdown files:

```
docs/plans/YYYY-MM-DD-pbi-NNN-short-description.md
```

Example: `docs/plans/2026-02-28-pbi-042-user-email-validation.md`

## Task Tracking

Use TodoWrite to maintain a task checklist as you write the plan. This helps you (and the human) track which tasks are defined and which need refinement:

```
- [ ] Task 1: Create user schema type
- [ ] Task 2: Write validation tests (red)
- [ ] Task 3: Implement validation (green)
- [ ] Task 4: Write API handler tests (red)
- [ ] Task 5: Implement API handler (green)
```

The TodoWrite checklist is your working memory during planning. The saved plan file is the deliverable.

## Worktree Consideration

If the plan will be executed in a separate session or by a parallel agent, note at the top of the plan whether it should run in a worktree:

```
Execution: worktree (isolated branch, merge to main when complete)
```

or

```
Execution: main (trunk-based, commit directly to main)
```

This tells the executor how to set up their environment before starting. See `skills/git-worktrees/` for worktree management.

## Codebase Exploration Checklist

Before writing the plan, verify you have explored:

- [ ] **Existing test patterns** — how are tests structured? What framework? Where do test files live?
- [ ] **File naming conventions** — how are files and directories named in this project?
- [ ] **Import patterns** — how does the project handle imports/modules?
- [ ] **Configuration** — where do configs live? Environment variables? Config files?
- [ ] **Similar features** — is there an existing feature similar to what you're planning? Follow its patterns.

Use Grep and Glob to answer these. Do not guess.

## Handoff to Execution

Once the plan is written and the human approves it:

1. **Same session:** The human says "execute the plan" or similar. Switch to the executing-plans skill.
2. **New session:** The human starts a fresh Claude Code session, provides the plan file path, and invokes the executing-plans skill.
3. **Subagent:** Use TaskCreate to spawn a subagent with the plan file path and instruction to follow the executing-plans skill.
4. **Parallel agents:** For plans with independent task groups, spawn multiple subagents via TaskCreate, each with their assigned tasks and a worktree.

## Example Interaction

```
Human: Plan the implementation for PBI #42 (email validation for the user registration form)

Agent: [Reads PBI #42, explores codebase, finds existing patterns]
       [Writes plan to docs/plans/2026-02-28-pbi-042-email-validation.md]
       [Presents summary to human for review]