# Finishing a Development Branch: Clean Completion of Branched Work

## Overview

Every development branch ends. The question is whether it ends with an explicit decision or quietly rots until someone notices it six months later.

**Core principle:** Every branch gets an explicit ending — merge, PR, keep, or discard. No branch left lingering.

**Violating the letter of this process is violating the spirit of clean version control.**

## Working Agreement

```
NO BRANCH LEFT BEHIND — EVERY BRANCH ENDS WITH AN EXPLICIT DECISION
```

A branch without an explicit ending is not "in progress" — it is abandoned. Abandoned branches are technical debt with a human cost: someone will waste time figuring out what happened.

## The 4 Options

When work on a branch is complete (or abandoned), exactly one of four options applies. Pick one explicitly.

| Option | When to Use | Result |
|--------|-----------|--------|
| **Merge locally** | Work is done, tests pass, no external review needed | Branch merged to base, branch deleted |
| **Push + PR** | External review needed, or team policy requires PRs | Branch pushed to remote, PR created, awaiting review |
| **Keep as-is** | Work is paused intentionally, will resume later | Branch stays, pause is documented |
| **Discard** | Experiment failed, work abandoned, branch is worthless | Branch deleted, worktree removed |

There is no fifth option. "I'll figure it out later" is not an option — it is a branch left behind.

### Option 1: Merge Locally

**Prerequisites:**
1. All changes committed (no uncommitted work)
2. Tests pass on the branch
3. Code review completed (if required by DoD)

**Process:**

```
1. Switch to the base branch
2. Merge the feature branch
3. Run the FULL test suite on the merged result
4. If tests pass → delete the feature branch
5. If tests fail → fix on base branch, do not go back to the feature branch
```

**Critical:** Test on the merged result, not just on the feature branch. The merge itself can introduce issues — conflict resolutions, interactions between changes, diverged dependencies. Individual branch tests passing is necessary but not sufficient.

### Option 2: Push + PR

**Prerequisites:**
1. All changes committed
2. Tests pass on the branch
3. Branch is ready for review (not WIP)

**Process:**

```
1. Push the branch to the remote with tracking (-u flag)
2. Create a Pull Request with:
   - Clear title describing the change
   - Description of what was implemented and why
   - Test results as evidence
   - Link to PBI/task if applicable
3. Wait for review (see skills/requesting-code-review/)
4. After PR is merged → delete the local branch and worktree
```

Do not push WIP branches for PR. If the work is not ready for review, it is not ready for a PR. Use Option 3 (Keep) instead.

### Option 3: Keep As-Is

**Prerequisites:**
1. All current work committed (no uncommitted changes)
2. A clear reason for pausing

**Process:**

```
1. Commit any remaining work with a descriptive message
2. Document why the branch is paused:
   - In the last commit message: "WIP: paused because [reason]"
   - OR in a branch description: git branch --edit-description
3. Set a reminder or create a PBI to resume the work
```

A kept branch without documented context is an abandoned branch waiting to happen. "I'll remember" is not documentation.

### Option 4: Discard

**Prerequisites:**
1. The branch has no value — experiment failed, approach abandoned, work superseded

**Process:**

```
1. Verify there is nothing worth salvaging (read the diff one last time)
2. Require TYPED CONFIRMATION of the branch name (not y/n)
3. Delete the branch: git branch -D <branch-name>
4. Remove the worktree if applicable: git worktree remove <path>
```

**Typed confirmation for destructive discard:** Before discarding a branch with uncommitted or unmerged work, the human must type the full branch name to confirm. This is not a y/n prompt — typing the full name forces conscious acknowledgment. This prevents accidental deletion of work that looked expendable at 2 AM but was actually valuable.

## Base Branch Auto-Detection

When merging, you need to know the base branch. Auto-detect it:

```
1. Check for 'main' branch         → git branch --list main
2. If not found, check 'master'    → git branch --list master
3. If neither, check git config    → git config init.defaultBranch
4. If all else fails               → ASK the human partner
```

Do not guess. Do not assume `main` exists. The auto-detection sequence exists because repositories vary and wrong assumptions cause merges to the wrong branch.

## Test Verification on Merged Result

This deserves its own section because it is the most commonly skipped step.

```
AFTER merging to base branch:

1. Run the FULL test suite (not just the tests you wrote)
2. Run the linter (if the project has one)
3. Run the build (if applicable)
4. Check for new warnings

ALL must pass. If any fail, fix on the base branch.
Do NOT go back to the feature branch — it no longer exists (or should not).
```

Why this matters: two branches can each pass all tests independently and fail when merged. Conflict resolutions can be subtly wrong. Dependencies can diverge. Interface contracts can change. The only way to know the merge is safe is to test the merge.

## Worktree Cleanup Rules

If the branch was worked on in a git worktree, cleanup includes the worktree itself:

| Ending | Worktree Action | Branch Action |
|--------|----------------|---------------|
| Merge locally | `git worktree remove <path>` | `git branch -d <branch>` |
| Push + PR | Keep until PR is merged, then remove | Delete after PR merge |
| Keep as-is | Worktree stays with the branch | Branch stays, documented |
| Discard | `git worktree remove <path>` | `git branch -D <branch>` |

After any worktree removal, run `git worktree prune` to clean up stale entries.

## Red Flags

**Stop and reconsider if you notice:**

- A branch existing for more than a week without activity — it is abandoned, not paused
- Merging without running tests on the merged result — "it worked on the branch" is not sufficient
- Discarding without typed confirmation — accidental deletion is permanent
- Leaving worktrees around after branches are merged — disk space and confusion accumulate
- "I'll get to it later" — lingering branches are forgotten branches
- Multiple branches with similar names — naming confusion causes wrong-branch merges
- A branch that has diverged significantly from base — rebase or merge base into the branch before merging back
- Pushing a WIP branch as a PR — a PR signals "ready for review"

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "I'll merge it later" | Schedule it now or it becomes abandoned |
| "I'll get to it later" | No you won't. Decide now. |
| "It's just one branch" | Abandoned branches compound. One becomes five becomes twenty. |
| "The tests passed on the branch" | Test the merge. Branch tests are not merge tests. |
| "I know what I'll break" | Run the tests. Intuition is not evidence. |
| "I don't need confirmation to discard" | Typed confirmation exists because you'll regret it without it. |
| "I'll clean up the worktrees eventually" | Clean up now. "Eventually" means never. |

## Connection to Other Skills

| Skill | Relationship |
|-------|-------------|
| `skills/git-worktrees/` | Worktree lifecycle — this skill handles the end of that lifecycle |
| `skills/verification/` | Test verification required after merge. Evidence before claiming the merge is safe. |
| `skills/parallel-agents/` | Parallel work produces multiple branches that all need finishing |
| `skills/requesting-code-review/` | Push+PR option triggers the review request process |
| `skills/code-review/` | Review must complete before merge (per DoD) |
| `skills/receiving-code-review/` | After PR review, respond to feedback, then finish the branch |

## Connection to SCRUM.md

A PBI is not done until its branch is finished. "Code is written" is not done. "Branch is merged, tests pass on main, worktree is cleaned up" is done. The branch finishing step is part of the Definition of Done, even if not explicitly listed — because unmerged code is undelivered code.

## Connection to DEVOPS.md

This skill implements the branch lifecycle described in DEVOPS.md's branching strategy:

```
main ──●──────────────●── (stable, tested, deployed)
        \            /
feature  ●──●──●──●─┘    (finished: merged + verified + cleaned up)
```

The merge arrow is not automatic. It passes through the Test Gate (run full suite), the Verification Gate (evidence of correctness), and the Review Gate (code review completed). This skill ensures all three gates are satisfied before the branch ends.

## The Bottom Line

**Every branch gets an explicit ending.**

Merge it, PR it, document the pause, or discard it with confirmation. No branch left behind, no worktree left dangling, no decision deferred.
