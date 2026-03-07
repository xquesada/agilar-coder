---
name: using-git-worktrees
description: Use when starting feature work that needs isolation or before executing implementation plans
---

# Using Git Worktrees — Claude Code Implementation

This is the Claude Code implementation of the git worktrees skill. The canonical process definition is in `skills/git-worktrees/SKILL.md`. Read it first — this file adds Claude Code-specific execution details, not a separate process.

## Claude Code Worktree Support

Claude Code has built-in worktree support via the **EnterWorktree** tool. This is the preferred method for creating worktrees in Claude Code sessions.

### Using EnterWorktree

The **EnterWorktree** tool:
- Creates a new git worktree inside `.claude/worktrees/` with a new branch based on HEAD
- Switches the session's working directory to the new worktree
- On session exit, prompts to keep or remove the worktree

Use it when:
- Starting feature work that should be isolated from `main`
- Before executing an implementation plan (keep `main` clean)
- When the user explicitly asks to work in a worktree

```
EnterWorktree(name: "feat-auth")
→ Creates .claude/worktrees/feat-auth/ with branch feat-auth
→ Session CWD moves to the new worktree
```

### Manual Worktree Creation

For finer control (custom base branch, custom directory), use **Bash** with git commands:

```bash
# Check existing worktrees
git worktree list

# Create worktree with specific base branch
git worktree add .worktrees/feat-auth -b feat-auth main

# Navigate to worktree
cd .worktrees/feat-auth
```

Use manual creation when:
- You need a base branch other than HEAD
- You want worktrees in a custom directory (not `.claude/worktrees/`)
- You are dispatching sub-agents that each need their own worktree

## Step-by-Step Process

### Step 1: Check for Existing Worktrees

```bash
git worktree list
```

Use **Bash** to run this before creating a new worktree. Avoid duplicates.

### Step 2: Verify Gitignore

Use **Read** to check `.gitignore` for the worktree directory. If using **EnterWorktree**, `.claude/worktrees/` should already be covered by `.claude/` in `.gitignore`. If using a custom directory, verify its entry exists.

If missing, use **Edit** to add the directory to `.gitignore` before proceeding.

### Step 3: Create the Worktree

**Option A — EnterWorktree (preferred for current session):**
```
EnterWorktree(name: "descriptive-name")
```

**Option B — Bash (for sub-agents or custom setup):**
```bash
git worktree add .worktrees/descriptive-name -b descriptive-name main
```

### Step 4: Project Setup

Use **Bash** to run setup commands in the new worktree:

```bash
# Detect and run appropriate setup
cd /path/to/worktree

# Node.js projects
[ -f package.json ] && npm install

# Go projects
[ -f go.mod ] && go mod download

# Python projects
[ -f requirements.txt ] && pip install -r requirements.txt
```

If using **EnterWorktree**, the session is already in the worktree directory. Run setup directly.

### Step 5: Verify Clean Baseline

Use **Bash** to run the test suite:

```bash
# Run project-specific test command
npm test          # Node.js
go test ./...     # Go
pytest            # Python
```

If tests fail before any changes, stop and report. Do not build on a broken baseline.

### Step 6: Report

Report worktree status. If you are the orchestrator dispatching to sub-agents, include this in the dispatch prompt so each agent knows its workspace.

## Dispatching Sub-Agents to Worktrees

When using `skills/parallel-agents/` with worktrees, create worktrees first, then include the worktree path in each sub-agent's prompt:

```bash
# Create worktrees for parallel agents
git worktree add .worktrees/agent-1-task -b agent-1-task main
git worktree add .worktrees/agent-2-task -b agent-2-task main
```

Then dispatch via **Agent** tool:

```
Agent 1 prompt:
"You are working in the worktree at /absolute/path/.worktrees/agent-1-task/
Your branch is agent-1-task. All file operations should use this directory.
[rest of task prompt]"

Agent 2 prompt:
"You are working in the worktree at /absolute/path/.worktrees/agent-2-task/
Your branch is agent-2-task. All file operations should use this directory.
[rest of task prompt]"
```

After both agents complete, merge from the main worktree:

```bash
cd /path/to/main/repo
git merge agent-1-task
git merge agent-2-task
npm test  # or equivalent — run full suite after merge
```

## Cleanup

```bash
# Remove a worktree after its branch is merged
git worktree remove .worktrees/feat-auth

# Delete the branch if merged
git branch -d feat-auth

# Prune stale entries
git worktree prune

# List all worktrees
git worktree list
```

If you used **EnterWorktree**, Claude Code prompts for cleanup on session exit.

## What NOT to Do

- Do not use **EnterWorktree** if you are already in a worktree. Claude Code prevents this, but be aware.
- Do not skip the gitignore check for custom worktree directories. `.claude/worktrees/` is covered by `.claude/` in standard setups, but `.worktrees/` is not covered by default.
- Do not forget `npm install` or equivalent in the new worktree. It is a fresh checkout — dependencies are not shared.
- Do not dispatch sub-agents to worktrees without giving them the absolute path. Sub-agents have no context about the worktree setup.
- Do not merge worktree branches without running the full test suite on the merge result. Individual worktree tests passing does not guarantee they work together.
