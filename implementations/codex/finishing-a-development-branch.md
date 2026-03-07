---
name: finishing-a-development-branch
description: Use when work on a branch or worktree is complete and you need to decide how to finish it — merge locally, push for PR, keep as-is, or discard. Covers cleanup, post-merge verification, and typed confirmation for destructive actions.
---

# Finishing a Development Branch — Codex Implementation

> Canonical skill: `skills/finishing-a-development-branch/SKILL.md`
>
> This file adds Codex-specific workflow. The working agreement, 4 options, worktree cleanup rules, and red flags in the canonical skill apply without modification. Read that file first.

## Working Agreement

```
NO BRANCH LEFT BEHIND — EVERY BRANCH ENDS WITH AN EXPLICIT DECISION
```

When work on a branch is complete, pick one of the 4 options and execute it. Do not leave branches lingering.

## Branch Finishing Protocol

### Step 0: Assess the Current State

Before choosing an option, understand where you are:

```bash
# What branch am I on?
git branch --show-current

# Are there uncommitted changes?
git status

# What commits are on this branch but not on the base?
git log --oneline main..HEAD
# or: git log --oneline master..HEAD

# Is there a worktree for this branch?
git worktree list

# Do all tests pass?
npm test        # or: go test ./... or: pytest or: cargo test
```

Run all of these before deciding. The assessment determines which option is valid.

### Base Branch Auto-Detection Script

```bash
# Auto-detect the base branch
if git branch --list main | grep -q main; then
    BASE_BRANCH="main"
elif git branch --list master | grep -q master; then
    BASE_BRANCH="master"
elif git config init.defaultBranch > /dev/null 2>&1; then
    BASE_BRANCH=$(git config init.defaultBranch)
else
    echo "Cannot auto-detect base branch. Ask the human partner."
    exit 1
fi
echo "Base branch: $BASE_BRANCH"
```

Use this before any merge operation. Do not hardcode `main`.

## Option 1: Merge Locally

Use when work is done, tests pass, and no external review is needed.

```bash
# 1. Ensure everything is committed
git status
# If uncommitted changes exist, commit them first

# 2. Verify tests pass on the branch
npm test        # or: go test ./... or: pytest

# 3. Switch to base branch
git checkout $BASE_BRANCH

# 4. Pull latest changes (other agents/humans may have pushed)
git pull --rebase origin $BASE_BRANCH

# 5. Merge the feature branch
git merge feature-branch-name

# 6. Run FULL test suite on the merged result
npm test        # or: go test ./...

# 7. Run linter on the merged result
npm run lint    # or: golangci-lint run

# 8. Run build on the merged result (if applicable)
npm run build   # or: go build ./...

# 9. If all pass -> delete the feature branch
git branch -d feature-branch-name

# 10. If worktree exists -> remove it
git worktree remove worktrees/feature-branch-name 2>/dev/null
git worktree prune
```

**Critical:** Steps 6-8 (test on merged result) are not optional. Do not skip them because "tests passed on the branch." The merge itself can introduce failures.

## Option 2: Push + PR

Use when external review is needed or team policy requires PRs.

```bash
# 1. Ensure everything is committed
git status

# 2. Verify tests pass on the branch
npm test        # or: go test ./... or: pytest

# 3. Push the branch with tracking
git push -u origin feature-branch-name

# 4. Create the PR (using gh CLI)
gh pr create \
  --title "Brief description of change" \
  --body "$(cat <<'EOF'
## Summary
- What was implemented and why

## Test Results
[paste test output]

## Notes
- Trade-offs, edge cases, open questions
EOF
)"
```

After the PR is created, the review process begins (see `skills/requesting-code-review/`). After the PR is merged:

```bash
# Cleanup after PR merge
git checkout $BASE_BRANCH
git pull origin $BASE_BRANCH
git branch -d feature-branch-name

# Remove worktree if applicable
git worktree remove worktrees/feature-branch-name 2>/dev/null
git worktree prune
```

## Option 3: Keep As-Is

Use when work is paused intentionally and will resume later.

```bash
# 1. Commit any remaining work with a clear message
git add -A
git commit -m "WIP: paused because [specific reason]

Will resume when [specific condition].
Context: [anything needed to resume without re-reading everything]"

# 2. Optionally push for backup
git push -u origin feature-branch-name

# 3. Note: do NOT remove the worktree — it stays with the branch
```

After committing, create a PBI or note to track the paused work so it does not become abandoned.

## Option 4: Discard

Use when the experiment failed or the work has no value.

```bash
# 1. Review what will be lost (one last look)
git diff $BASE_BRANCH..HEAD
git log --oneline $BASE_BRANCH..HEAD

# 2. REQUIRE TYPED CONFIRMATION from the human partner
# Ask: "Type the full branch name to confirm discard: "
# Do NOT proceed on y/n — require the exact branch name

# 3. After confirmation — switch to base branch
git checkout $BASE_BRANCH

# 4. Remove worktree first (if applicable)
git worktree remove worktrees/feature-branch-name 2>/dev/null
git worktree prune

# 5. Force-delete the branch (it was never merged)
git branch -D feature-branch-name

# 6. Remove remote branch if it was pushed
git push origin --delete feature-branch-name 2>/dev/null
```

**Typed confirmation is mandatory.** Before running the destructive commands (steps 3-6), ask the human partner to type the full branch name. "y" or "yes" is not acceptable — the full name forces conscious acknowledgment.

## Post-Merge Verification Sequence

After merging (Option 1), run this complete verification sequence:

```bash
# On the base branch, after merge:

# 1. Full test suite
npm test
# Expect: all tests pass, 0 failures

# 2. Linter check
npm run lint
# Expect: 0 errors, 0 warnings (or only pre-existing warnings)

# 3. Build check
npm run build
# Expect: exit code 0

# 4. Git status — should be clean
git status
# Expect: nothing to commit, working tree clean
```

Report each result with actual output (see `skills/verification/`). Do not say "all good" — paste the evidence.

## Worktree Cleanup

If you used `git worktree add` to create the worktree:

- For Options 1 and 4, cleanup should happen as part of the finishing process
- For Option 2, the worktree stays until the PR is merged
- For Option 3, the worktree stays with the paused branch

After finishing via Options 1 or 4, navigate back to the main repo directory:

```bash
cd /path/to/main/repo
```

## Decision Flow

When you reach the end of branch work, follow this decision tree:

```
Is the work complete?
+-- YES -> Do tests pass?
|   +-- YES -> Is external review needed?
|   |   +-- YES -> Option 2: Push + PR
|   |   +-- NO  -> Option 1: Merge locally
|   +-- NO  -> Fix tests first, then re-evaluate
+-- PAUSED -> Option 3: Keep as-is (document why)
+-- ABANDONED -> Option 4: Discard (typed confirmation required)
```

## What NOT to Do

- Do not leave branches without an explicit decision. "I will figure it out later" means the branch is abandoned.
- Do not merge without running the full test suite on the merged result. Branch tests passing is not merge tests passing.
- Do not discard without typed confirmation from the human partner. `git branch -D` is permanent.
- Do not forget worktree cleanup. After merge or discard, remove the worktree and prune stale entries.
- Do not push WIP branches as PRs. A PR signals "ready for review." If it is not ready, use Option 3 (Keep).
- Do not skip `git pull --rebase` before merging to base. Other agents or humans may have pushed changes. Merging into a stale base creates unnecessary conflicts.
- Do not delete branches that have unmerged work without the discard confirmation process. Use `git branch -d` (lowercase) for merged branches — it will refuse to delete unmerged branches. Use `git branch -D` (uppercase) only after typed confirmation.
