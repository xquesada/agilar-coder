---
name: sprint-planning
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Sprint Planning — Claude Code Implementation

This implements `skills/sprint-planning/SKILL.md` for Claude Code.

## Entering Plan Mode

When the human asks you to plan (or when a PBI is complex enough to warrant a plan), switch into plan mode:

1. **Read the PBI.** Understand the goal, acceptance criteria, and any design decisions already made.
2. **Explore the codebase.** Use Read, Grep, and Glob to understand existing patterns, conventions, file structure, and test infrastructure. The plan must fit the codebase as it is, not as you imagine it.
3. **PBI size check.** Before drafting the plan, estimate the number of tasks. If you can already see the PBI would need more than 5-7 tasks (10-15 minutes of agent execution), stop and suggest splitting. Present the split options to the human — they decide whether to split or proceed as-is.
4. **Draft the plan.** Follow the structure from `skills/sprint-planning/SKILL.md` — header with Goal/Architecture/Tech Stack/Acceptance Criteria, then numbered tasks at 2-5 minute granularity.
5. **Save the plan in the PBI file.** Write it as a `## Plan` section inside the PBI file. Plans are artifacts — they persist across sessions.

## Plan File Convention

Plans are written as the `## Plan` section of the PBI file:

```
backlog/pbi-NNN-short-description.md
```

Example: `backlog/pbi-042-email-validation.md` (or `backlog/ready/pbi-042-email-validation.md` if approved in the same session).

Design docs (brainstorming output) also go in the PBI file under a `## Design` section. If a design applies to multiple PBIs, write it in the first PBI and reference it from the others.

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

## Full Suite Before Every Commit

Every task's commit step must run the project's **full** test and lint commands — not just the package-level tests. Read the project's CLAUDE.md for the exact commands (typically `make test && make lint` or equivalent).

In trunk-based development, every commit lands on `main`. A package-level green is not enough — cross-package breakages must be caught before commit. The task template should include a **Pre-commit** step:

```
**Pre-commit:**
  Run: make test && make lint
  Expected: all green
```

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

Once the plan is written and the human approves it, move the PBI file to `backlog/ready/`:

```bash
mv backlog/pbi-NNN-*.md backlog/ready/
```

Then the execution approach depends on team mode. **You decide — do not ask the user to choose between execution strategies.**

### Solo Mode (default)

Count the tasks in the plan and recommend accordingly:

- **Up to ~8 tasks:** Execute in the current session. Just say: "Plan ready. Want me to start?" Move the PBI to `backlog/ready/`, then switch to the executing-plans skill.
- **9+ tasks:** Recommend a fresh session. Say: "This plan has N tasks. I recommend starting a fresh session to execute it — that way I won't lose context midway. The PBI file is at `backlog/ready/pbi-NNN-description.md`. Start a fresh session and tell it to build the next ready PBI."

Do not mention sub-agents, context windows, or compaction to the user. Just make the recommendation.

### Multi-Agent Mode

Dispatch independent task groups to parallel agents in worktrees. Plans are queued in `backlog/ready/`. The orchestrator dispatches workers. See `skills/parallel-agents/` and `skills/git-worktrees/`.

### Multi-Human Mode

Assign task groups to developers. Each works in a feature branch and opens a PR when complete.

## Example Interaction

```
Human: Plan the implementation for PBI #42 (email validation for the user registration form)

Agent: [Reads PBI #42, explores codebase, finds existing patterns]
       [Creates/updates backlog/pbi-042-email-validation.md with description, AC, and Plan section]
       [Presents summary to human for review]