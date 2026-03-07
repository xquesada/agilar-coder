---
name: using-git-worktrees
description: Use when starting feature work that needs isolation or before executing implementation plans. Creates a separate working directory with its own branch.
---

<!-- Adapted from Claude Code: EnterWorktree tool replaced with git worktree add commands. Default directory changed from .claude/worktrees/ to .worktrees/. Sub-agent dispatch section removed (Codex executes in one session). -->

# Using Git Worktrees — Codex Implementation

> Canonical skill: `skills/git-worktrees/SKILL.md`
>
> This file adapts the git worktrees skill for Codex. All worktree operations use standard `git worktree` commands.

## When to Use Worktrees

- Starting feature work that should be isolated from `main`
- Before executing an implementation plan (keep `main` clean)
- When the user explicitly asks to work in a worktree
- When running multiple Codex sessions in parallel (each gets its own worktree)

## Step-by-Step Process

### Step 1: Check for Existing Worktrees

Run in terminal:

```bash
git worktree list
```

Avoid creating duplicates.

### Step 2: Verify Gitignore

Read `.gitignore` and check for the `.worktrees/` directory entry. If missing, add it before proceeding:

```bash
echo '.worktrees/' >> .gitignore
```

### Step 3: Create the Worktree

```bash
# Default: branch from current HEAD
git worktree add .worktrees/feat-auth -b feat-auth

# From a specific base branch
git worktree add .worktrees/feat-auth -b feat-auth main
```

### Step 4: Project Setup

Run setup commands in the new worktree:

```bash
cd /path/to/repo/.worktrees/feat-auth

# Node.js projects
[ -f package.json ] && npm install

# Go projects
[ -f go.mod ] && go mod download

# Python projects
[ -f requirements.txt ] && pip install -r requirements.txt
```

### Step 5: Verify Clean Baseline

Run the test suite in the new worktree:

```bash
cd /path/to/repo/.worktrees/feat-auth

# Run project-specific test command
npm test          # Node.js
go test ./...     # Go
pytest            # Python
```

If tests fail before any changes, stop and report. Do not build on a broken baseline.

### Step 6: Do the Work

Work in the worktree directory. All file reads, edits, and terminal commands should target the worktree path.

## Merging Back

After work is complete and all tests pass in the worktree:

```bash
# Switch to main repo
cd /path/to/main/repo

# Merge the feature branch
git merge feat-auth

# Run full test suite on merged result
npm test  # or equivalent
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

## What NOT to Do

- Do not skip the gitignore check for `.worktrees/`. It is not covered by default.
- Do not forget `npm install` or equivalent in the new worktree. It is a fresh checkout — dependencies are not shared.
- Do not merge worktree branches without running the full test suite on the merge result. Individual worktree tests passing does not guarantee they work together.
- Do not leave stale worktrees behind. Clean up after merging.
