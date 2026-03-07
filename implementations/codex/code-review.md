---
name: code-review
description: Use when completing tasks, implementing features, or before merging to verify work meets requirements. Triggers a two-stage review — spec compliance first, then code quality.
---

# Code Review — Codex Implementation

> Canonical skill: `skills/code-review/SKILL.md`

This is the Codex implementation of the code review skill. The canonical process definition is in `skills/code-review/SKILL.md`. Read it first — this file adds Codex-specific execution details, not a separate process.

## How to Perform a Review

Codex does not have subagent dispatch. Instead, perform both review stages sequentially yourself, treating each stage as a distinct pass with a distinct focus. Do not blend the two stages.

### 1. Gather the diff

```bash
# For reviewing work since a specific commit
BASE_SHA=$(git merge-base HEAD main)
HEAD_SHA=$(git rev-parse HEAD)
git diff --stat $BASE_SHA..$HEAD_SHA
git diff $BASE_SHA..$HEAD_SHA
```

### 2. Stage 1: Spec Compliance Review

Read the requirements (PBI acceptance criteria, design doc, or task description). Then read the actual changed files and compare against the requirements line by line.

**Check for:**

**Missing requirements:**
- Did the implementation cover everything that was requested?
- Are there requirements that were skipped or missed?
- Does anything claim to work but lack actual implementation?

**Extra/unneeded work:**
- Was anything built that was not requested?
- Was anything over-engineered or unnecessarily added?

**Misunderstandings:**
- Were requirements interpreted differently than intended?
- Was the right feature built the wrong way?

**How to verify:**
- Read the changed files and search for patterns referenced in the requirements
- Run tests to confirm claimed behavior actually works

**Output for Stage 1:**
- **PASS:** Spec compliant (all requirements verified in code)
- **FAIL:** Issues found — list specifically what is missing, extra, or misunderstood, with file:line references

**Only proceed to Stage 2 if Stage 1 passes.** If the implementation does not match the spec, code quality review is wasted effort.

### 3. Stage 2: Code Quality Review

Read the changed files again, this time focusing on production readiness.

**Review Checklist:**

**Code Quality:**
- Clean separation of concerns?
- Proper error handling?
- Type safety (if applicable)?
- DRY principle followed?
- Edge cases handled?

**Architecture:**
- Sound design decisions?
- Scalability considerations?
- Performance implications?
- Security concerns?

**Testing:**
- Tests actually test logic (not mocks testing mocks)?
- Edge cases covered?
- Integration tests where needed?
- All tests passing?
- TDD discipline followed?

**Production Readiness:**
- Migration strategy (if schema changes)?
- Backward compatibility considered?
- Documentation complete?
- No obvious bugs?

**Output for Stage 2:**

### Strengths
[What is well done. Be specific — cite files and lines.]

### Issues

#### Critical (Must Fix)
[Bugs, security issues, data loss risks, broken functionality]

#### Important (Should Fix)
[Architecture problems, missing features, poor error handling, test gaps]

#### Minor (Nice to Have)
[Code style, optimization opportunities, documentation improvements]

For each issue include: file:line reference, what is wrong, why it matters, how to fix.

### Assessment

**Verdict:** [Ready to merge / Ready with fixes / Not ready]

**Reasoning:** [1-2 sentence technical assessment]

### 4. Act on feedback

- Fix Critical issues immediately
- Fix Important issues before proceeding
- Note Minor issues for later
- Push back if a finding is wrong — with reasoning

## Integration with Workflows

**After each task or PBI:**
- Gather the diff from the start of the task
- Run Stage 1 (spec compliance)
- If Stage 1 passes, run Stage 2 (code quality)
- Fix issues before moving to the next task

**Before merge to main:**
- Gather the full diff from `origin/main`
- Run both stages
- All Critical and Important issues must be resolved

## What NOT to Do

- Do not skip Stage 1 (spec compliance). Building the wrong thing well is still building the wrong thing.
- Do not self-review by just re-reading your own code. Treat each stage as a fresh, independent pass — focus only on that stage's checklist.
- Do not bundle multiple tasks into one review. Review each task separately so feedback is specific and actionable.
- Do not ignore Important issues. The severity level reflects the impact, not the effort.
- Do not perform a review without the requirements. A reviewer without a spec can only check style, not correctness.
