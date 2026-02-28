# Git Worktrees: Isolated Workspaces for Parallel Work

Create and manage isolated git worktrees so multiple agents (or the same agent working on different tasks) can operate on the same repository simultaneously without filesystem conflicts. Each worktree is a separate working directory with its own branch, sharing the same git history.

This skill is primarily for multi-agent team mode (see `SCRUM.md`), where multiple agents implement different PBIs or tasks in parallel. It can also be used in solo mode for isolating risky or experimental work.

## What Is a Git Worktree?

A git worktree is a linked checkout of the same repository in a different directory. Unlike `git clone`, worktrees share the object database and refs — there is no duplication. Unlike branches, worktrees provide filesystem isolation — each has its own working directory, index, and HEAD.

```
repo/                     ← main worktree (main branch)
repo/.worktrees/feat-a/   ← worktree for feature A (feat-a branch)
repo/.worktrees/feat-b/   ← worktree for feature B (feat-b branch)
```

All three share the same `.git` directory. Commits in one are immediately visible to the others. But file changes in one do not appear in the others until committed and merged.

## When to Use

- **Multi-agent parallel implementation** — each agent gets its own worktree, works independently, orchestrator merges results to `main`
- **Isolating risky changes** — test a dangerous refactor without affecting the main working directory
- **Context switching** — work on task A in one worktree, switch to task B in another, without stashing or committing incomplete work
- **Before executing implementation plans** — create a worktree for the implementation to keep `main` clean until all tasks are verified

## When NOT to Use

- **Solo trunk-based development** — if you commit directly to `main` and never work in parallel, worktrees add no value
- **Quick single-file changes** — the overhead of creating a worktree outweighs the benefit for trivial changes
- **Read-only investigation** — multiple agents can read from the same working directory without conflict. Worktrees are for write isolation.

## Process

### Step 1: Choose the Worktree Directory

Determine where worktrees will live. The convention:

1. **Check for existing worktrees** — run `git worktree list` to see what already exists. Do not create duplicates.
2. **Check project configuration** — look for a `.worktrees/` directory or similar convention in the project. If one exists, use it.
3. **If no convention exists** — use `<repo-root>/.worktrees/` as the default location. Create the directory if needed.
4. **If uncertain** — ask the human partner where worktrees should live.

### Step 2: Verify the Directory Is Gitignored

**Critical safety check.** The worktree directory must be in `.gitignore`. If it is not, worktree contents will appear as untracked files in the main worktree, causing confusion and accidental commits.

Check `.gitignore` for an entry matching the worktree directory (e.g., `.worktrees/`). If missing, add it before proceeding.

Do not skip this step. Do not assume it is already there. Verify.

### Step 3: Create the Worktree

Create a new worktree with a new branch based on the current state of `main` (or whichever base branch is appropriate):

```
git worktree add <path> -b <branch-name> <base-branch>
```

Example:
```
git worktree add .worktrees/feat-auth -b feat-auth main
```

This creates:
- A new directory `.worktrees/feat-auth/` with a full working copy
- A new branch `feat-auth` based on `main`
- The worktree is immediately ready for work

Branch naming convention: use descriptive names that match the task or PBI. For agent-driven work, include the agent identifier if multiple agents are active (e.g., `agent-1-feat-auth`).

### Step 4: Run Project Setup

The worktree is a fresh checkout — it may need dependency installation or build steps before it is usable.

Auto-detect the project type and run setup:

| Indicator File | Setup Command |
|---------------|---------------|
| `package.json` | `npm install` (or `yarn install`, `pnpm install`) |
| `go.mod` | `go mod download` |
| `requirements.txt` | `pip install -r requirements.txt` |
| `Gemfile` | `bundle install` |
| `Cargo.toml` | `cargo build` |
| `pom.xml` | `mvn install` |
| `Makefile` with `setup` target | `make setup` |

If multiple indicators exist, use the primary one (typically the one in the project root). If unsure, ask the human partner.

### Step 5: Verify Clean Baseline

Before any work begins in the worktree, verify that the starting state is clean:

1. **Run the full test suite** — all tests must pass in the fresh worktree. If tests fail before any changes, there is a problem with `main` or with the setup. Do not proceed with failing baseline tests.
2. **Check git status** — the only changes should be generated files from setup (e.g., `node_modules/`, build artifacts). These should already be gitignored.

If baseline tests fail, stop. Report the failure. Do not start work on top of a broken baseline.

### Step 6: Report

After setup, report the worktree status to the orchestrator or human partner:

- Worktree path
- Branch name
- Base branch and commit
- Setup commands run
- Test suite status (all passing / N failures)
- Ready for work: yes/no

## Working in a Worktree

Once created, the worktree behaves like a normal git checkout:

- Make changes, commit, push — all standard git operations work
- The branch is specific to this worktree — commits here do not affect `main` or other worktrees
- Other worktrees and the main working directory are unaffected by changes here

### Committing and Merging

When work in a worktree is complete:

1. Commit all changes to the worktree's branch
2. Switch to the main worktree (or the orchestrator does this)
3. Merge the worktree's branch into `main`
4. Run the full test suite on `main` after merge
5. Remove the worktree if no longer needed (see Cleanup)

In multi-agent mode, the orchestrator handles steps 2-5. Individual agents commit to their branch and report completion.

## Cleanup

Remove a worktree when its work is merged and no longer needed:

```
git worktree remove <path>
```

This removes the working directory and the worktree registration. The branch remains (delete it separately with `git branch -d <branch>` if merged).

To list all worktrees and their status:
```
git worktree list
```

Prune stale worktree entries (e.g., if the directory was manually deleted):
```
git worktree prune
```

## Red Flags

- **Skipping the gitignore verification** — this causes worktree contents to show up as untracked files in the main worktree. Always verify before creating.
- **Assuming the worktree directory exists** — check first, create if needed. Do not error out on a missing directory.
- **Proceeding with failing baseline tests** — if tests fail in a fresh worktree before any changes, `main` is broken. Fix `main` first.
- **Forgetting project setup** — a worktree is a fresh checkout. It needs `npm install` or equivalent before tests will run.
- **Multiple worktrees on the same branch** — git prevents this, but attempting it indicates a process problem. One worktree per branch.
- **Never cleaning up** — old worktrees accumulate. Remove them when merged. Run `git worktree list` periodically.

## Connection to Other Skills

| Skill | Relationship |
|-------|-------------|
| `skills/parallel-agents/` | Worktrees provide the filesystem isolation that makes parallel implementation safe |
| `skills/subagent-driven/` | Workers can operate in worktrees for isolation, though single-worktree execution is also fine |
| `skills/verification/` | Baseline test verification before starting work in a new worktree |

## Connection to DEVOPS.md

Git worktrees implement the multi-agent branching strategy described in `DEVOPS.md`:

```
main ──●──────────●──────●──
        \        / \    /
agent-1  ●──●──●    \  /
          \          \/
agent-2    ●──●──●──●
```

Each agent line is a worktree. The orchestrator merges completed worktrees to `main`, runs the full test suite (Test Gate), and verifies integration (Verification Gate). Worktrees are the mechanism — the branching strategy is the policy.
