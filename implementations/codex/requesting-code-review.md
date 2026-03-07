---
name: requesting-code-review
description: Use when you need to request a code review — after completing a task, before merging, or when you want a sanity check on complex changes. Covers gathering context, running pre-review checks, and structuring the review request.
---

# Requesting Code Review — Codex Implementation

> Canonical skill: `skills/requesting-code-review/SKILL.md`
>
> This file adds Codex-specific workflow. The working agreement, dispatch template, severity-based response table, and red flags in the canonical skill apply without modification. Read that file first.

## Working Agreement

```
NO REVIEW REQUEST WITHOUT A COMPLETE DISPATCH
```

Before requesting a review (self-review or human), every field in the dispatch template must be filled. Incomplete dispatches produce incomplete reviews.

## Codex Review Request Protocol

### Step 1: Gather the Diff

```bash
# Find the base commit (before your work started)
BASE_SHA=$(git merge-base HEAD main)
# or: BASE_SHA=$(git merge-base HEAD master)
# or: BASE_SHA=<specific-commit-hash>

# Find the head commit (your latest change)
HEAD_SHA=$(git rev-parse HEAD)

# View the summary of changes
git diff --stat $BASE_SHA..$HEAD_SHA

# View the full diff (for reference when filling the template)
git diff $BASE_SHA..$HEAD_SHA

# List commits in the range
git log --oneline $BASE_SHA..$HEAD_SHA
```

### Step 2: Run Tests (Pre-Review Verification)

```bash
# Run the full test suite
npm test        # or: go test ./... -v or: pytest -v

# Run the linter
npm run lint    # or: golangci-lint run or: ruff check .

# Run the build
npm run build   # or: go build ./... or: cargo build
```

All must pass before requesting a review. If any fail, fix first.

### Step 3: Build the Dispatch

Read the plan or PBI acceptance criteria file. Do not quote from memory — paste the actual requirements.

```bash
# If the plan is in a file, read it directly
cat docs/plans/plan-name.md

# If the plan is a PBI, fetch it
curl -s http://M3.local:8076/api/backlog/42 | jq '.notes, .checklist'
# (adapt URL to your project's backlog system)
```

### Step 4: Perform the Review

Since Codex does not have subagent dispatch, perform the two-stage review sequentially yourself (follow the `code-review` skill), or prepare a dispatch for a human reviewer.

#### Self-Review (Two Stages)

Follow the `code-review` skill instructions to perform:
1. **Stage 1: Spec Compliance** — compare implementation against requirements line by line
2. **Stage 2: Code Quality** — only if Stage 1 passes

Treat each stage as an independent pass. Do not blend them.

#### Requesting Human Review

When the reviewer is a human (via PR or conversation), format the request as a clear summary:

```
## Review Request

**What was implemented:** [1-2 sentence summary]

**Requirements (PBI #N):**
[Paste acceptance criteria]

**Changes:**
- `file1`: what changed and why
- `file2`: what changed and why

**Diff:** `git diff [BASE_SHA]..[HEAD_SHA]`

**Test results:**
```
[paste actual test output]
```

**Notes:**
- [trade-offs, edge cases, open questions]
```

### Step 5: Act on Review Results

After completing the review (self or human), follow `skills/receiving-code-review/`:

```bash
# For each accepted fix:
# 1. Make the fix
# 2. Run tests after fix
npm test

# After ALL fixes:
# Run full verification
npm test && npm run lint && npm run build
```

If the verdict is "Not ready" (Critical issues), fix and re-run the review:

```bash
# After fixing Critical issues:
NEW_HEAD_SHA=$(git rev-parse HEAD)
# Re-review with updated range: BASE_SHA stays the same, HEAD_SHA = NEW_HEAD_SHA
```

## Pre-Dispatch Checklist

Before requesting review, verify each item:

```bash
# [ ] All changes committed
git status
# Expect: nothing to commit

# [ ] Tests pass (full suite)
npm test
# Expect: 0 failures

# [ ] Linter passes
npm run lint
# Expect: 0 errors

# [ ] Build succeeds
npm run build
# Expect: exit code 0

# [ ] BASE_SHA and HEAD_SHA are correct
git merge-base HEAD main
git rev-parse HEAD

# [ ] Plan/requirements are available
# Read the plan file or fetch the PBI
```

If any check fails, fix before requesting review. Do not request a review on code that does not build, does not pass tests, or has uncommitted changes.

## What NOT to Do

- Do not request review without test results. Run the tests and paste the output. "Tests pass" without evidence is not a dispatch — it is a claim.
- Do not send incomplete dispatches. Every field in the template exists for a reason. Missing the PLAN field means the reviewer cannot check spec compliance. Missing TEST_RESULTS means the reviewer will waste time finding failures you should have caught.
- Do not request review on untested code. If you have not run the test suite since your last change, you are not ready for review.
- Do not skip the dispatch template because "the reviewer can figure it out." The 5 minutes of template preparation saves 30 minutes of reviewer confusion.
- Do not perform Stage 2 (code quality) before Stage 1 (spec compliance) passes. If the wrong thing was built, code quality is irrelevant.
- Do not request review on uncommitted changes. Commit first — the review is against a specific diff range, not the working directory.
- Do not make more changes after starting a review. The review is against BASE_SHA..HEAD_SHA. Changes after review start invalidate the review.
