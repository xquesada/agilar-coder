---
name: requesting-code-review
description: Use when you need to request a code review — after completing a task, before merging, or when you want a sanity check on complex changes
---

# Requesting Code Review — Claude Code Implementation

> Canonical skill: `skills/requesting-code-review/SKILL.md`
>
> This file adds Claude Code-specific tooling and workflow. The working agreement, dispatch template, severity-based response table, and red flags in the canonical skill apply without modification. Read that file first.

## Working Agreement

```
NO REVIEW REQUEST WITHOUT A COMPLETE DISPATCH
```

Before dispatching a reviewer (Agent tool or human), every field in the dispatch template must be filled. Incomplete dispatches produce incomplete reviews.

## Claude Code Review Request Protocol

You have the Bash tool for git commands, the Agent tool for dispatching reviewers, and the Read tool for gathering context. Use them to build a complete dispatch.

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

# View the full diff (for your reference when filling the template)
git diff $BASE_SHA..$HEAD_SHA

# List commits in the range
git log --oneline $BASE_SHA..$HEAD_SHA
```

### Step 2: Run Tests (Pre-Dispatch Verification)

```bash
# Run the full test suite — paste this output in the dispatch
npm test        # or: go test ./... -v or: pytest -v

# Run the linter
npm run lint    # or: golangci-lint run or: ruff check .

# Run the build
npm run build   # or: go build ./... or: cargo build
```

All must pass before dispatching a review. If any fail, fix first.

### Step 3: Build the Dispatch

Use the Read tool to re-read the plan or PBI acceptance criteria. Do not quote from memory — paste the actual requirements.

```bash
# If the plan is in a file:
# Use Read tool: Read docs/plans/plan-name.md

# If the plan is a PBI, fetch it:
# Use Bash: curl -s http://M3.local:8076/api/backlog/42 | jq '.notes, .checklist'
# (adapt URL to your project's backlog system)
```

### Step 4: Dispatch the Reviewer

#### Dispatching an Agent Reviewer (Stage 1: Spec Compliance)

Use the **Agent** tool. Fill every placeholder — no exceptions.

```
You are reviewing whether an implementation matches its specification.

## What Was Requested

[PASTE the full PBI acceptance criteria, design doc section, or task description.
Do not summarize. Paste the actual text.]

## What Was Implemented

[1-2 sentence summary of what was built]

## CRITICAL: Verify Independently

Do NOT trust summaries. You MUST verify by reading the actual code.

- Read the actual code changes
- Compare implementation to requirements line by line
- Check for missing pieces
- Look for extra work that was not requested

## Git Range

Base: [BASE_SHA]
Head: [HEAD_SHA]

```bash
git diff --stat [BASE_SHA]..[HEAD_SHA]
git diff [BASE_SHA]..[HEAD_SHA]
```

## Output

Report: PASS (spec compliant) or FAIL (issues found — list with file:line references)
```

#### Dispatching an Agent Reviewer (Stage 2: Code Quality)

Only dispatch if Stage 1 passes. Use the **Agent** tool:

```
You are a Senior Code Reviewer. Review for production readiness.

## What Was Implemented

[Description of what was built and why]

## Requirements

[Paste the requirements/plan]

## Git Range

Base: [BASE_SHA]
Head: [HEAD_SHA]

```bash
git diff --stat [BASE_SHA]..[HEAD_SHA]
git diff [BASE_SHA]..[HEAD_SHA]
```

## Review Checklist

Use Read and Grep tools to examine the actual code:

- Code quality (separation of concerns, error handling, DRY, edge cases)
- Architecture (design, scalability, security)
- Testing (real tests, edge cases, coverage)
- Production readiness (migrations, backward compat, docs)

## Output

### Strengths
[What is well done — cite files and lines]

### Issues
#### Critical (Must Fix)
#### Important (Should Fix)
#### Minor (Nice to Have)

For each issue: file:line, what is wrong, why it matters, how to fix.

### Assessment
**Verdict:** [Ready to merge / Ready with fixes / Not ready]
**Reasoning:** [1-2 sentence assessment]
```

#### Dispatching for Human Review

When the reviewer is a human (via PR or conversation), format the dispatch as a clear summary:

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

After receiving the review (from agent or human), follow `skills/receiving-code-review/`:

```bash
# For each accepted fix:
# 1. Make the fix
# 2. Run tests after fix
npm test

# After ALL fixes:
# Run full verification
npm test && npm run lint && npm run build
```

If the verdict is "Not ready" (Critical issues), fix and re-dispatch the review:

```bash
# After fixing Critical issues:
NEW_HEAD_SHA=$(git rev-parse HEAD)

# Re-dispatch with updated range
# BASE_SHA stays the same, HEAD_SHA = NEW_HEAD_SHA
```

## Integration with Subagent-Driven Workflow

When using `skills/subagent-driven/`, review dispatch is part of the task completion cycle:

```
For each task:
1. Worker completes task → commits
2. Orchestrator gathers dispatch data (diff, plan, test results)
3. Orchestrator dispatches Stage 1 reviewer (Agent tool)
4. If Stage 1 passes → dispatch Stage 2 reviewer (Agent tool)
5. Process review results (skills/receiving-code-review/)
6. Fix issues if any → re-verify → move to next task
```

The dispatch template is the same. The only difference is that the orchestrator fills it based on the worker's output, not their own work.

## Pre-Dispatch Checklist (Claude Code)

Before dispatching, verify each item with the actual tool:

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
# Use Read tool to read the plan file or fetch the PBI
```

If any check fails, fix before dispatching. Do not dispatch a review on code that does not build, does not pass tests, or has uncommitted changes.

## What NOT to Do

- Do not request review without test results. Run the tests and paste the output in the dispatch. "Tests pass" without evidence is not a dispatch — it is a claim.
- Do not send incomplete dispatches. Every field in the template exists for a reason. Missing the PLAN field means the reviewer cannot check spec compliance. Missing TEST_RESULTS means the reviewer will waste time finding failures you should have caught.
- Do not request review on untested code. If you have not run the test suite since your last change, you are not ready for review.
- Do not skip the dispatch template because "the reviewer can figure it out." The reviewer should not have to discover context you already have. The 5 minutes of template preparation saves 30 minutes of reviewer confusion.
- Do not dispatch Stage 2 (code quality) before Stage 1 (spec compliance) passes. If the wrong thing was built, code quality is irrelevant.
- Do not request review on uncommitted changes. Commit first — the review is against a specific diff range, not the working directory.
- Do not make more changes after dispatching a review. The review is against BASE_SHA..HEAD_SHA. Changes after dispatch invalidate the review.
