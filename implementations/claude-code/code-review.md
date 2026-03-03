---
name: code-review
description: Use when completing tasks, implementing features, or before merging to verify work meets requirements
---

# Code Review — Claude Code Implementation

This is the Claude Code implementation of the code review skill. The canonical process definition is in `skills/code-review/SKILL.md`. Read it first — this file adds Claude Code-specific execution details, not a separate process.

## How to Request a Review

### 1. Gather the diff

```bash
# For reviewing work since a specific commit
BASE_SHA=$(git rev-parse HEAD~N)  # or origin/main, or the commit before your work started
HEAD_SHA=$(git rev-parse HEAD)
git diff --stat $BASE_SHA..$HEAD_SHA
```

### 2. Dispatch a spec compliance reviewer (Stage 1)

Use the **Agent** tool with the spec reviewer template below. Fill in the placeholders.

**Only proceed to Stage 2 if Stage 1 passes.** If the implementation doesn't match the spec, code quality review is wasted effort.

### 3. Dispatch a code quality reviewer (Stage 2)

Use the **Agent** tool with the code reviewer template below. Fill in the placeholders.

### 4. Act on feedback

- Fix Critical issues immediately
- Fix Important issues before proceeding
- Note Minor issues for later
- Push back if the reviewer is wrong — with reasoning

## Agent Templates

### Spec Compliance Reviewer

Use this as the prompt for an **Agent** tool call:

```
You are reviewing whether an implementation matches its specification.

## What Was Requested

{FULL TEXT OF REQUIREMENTS — paste the PBI acceptance criteria, design doc section, or task description}

## What Was Implemented

{BRIEF DESCRIPTION of what was built}

## CRITICAL: Verify Independently

Do NOT trust summaries of what was implemented. You MUST verify everything by reading the actual code.

**DO NOT:**
- Take the developer's word for what they implemented
- Trust claims about completeness
- Accept their interpretation of requirements

**DO:**
- Read the actual code changes
- Compare actual implementation to requirements line by line
- Check for missing pieces
- Look for extra features that weren't requested

## Your Job

Read the implementation code and verify:

**Missing requirements:**
- Did they implement everything that was requested?
- Are there requirements they skipped or missed?
- Did they claim something works but didn't actually implement it?

**Extra/unneeded work:**
- Did they build things that weren't requested?
- Did they over-engineer or add unnecessary features?
- Did they add "nice to haves" that weren't in spec?

**Misunderstandings:**
- Did they interpret requirements differently than intended?
- Did they solve the wrong problem?
- Did they implement the right feature but wrong way?

## How to Review

Use Read and Grep tools to examine the changed files. Compare what you find against the requirements above.

## Git Range

Base: {BASE_SHA}
Head: {HEAD_SHA}

```bash
git diff --stat {BASE_SHA}..{HEAD_SHA}
git diff {BASE_SHA}..{HEAD_SHA}
```

## Output

Report one of:
- PASS: Spec compliant (if everything matches after code inspection)
- FAIL: Issues found — list specifically what's missing, extra, or misunderstood, with file:line references
```

### Code Quality Reviewer

Use this as the prompt for an **Agent** tool call:

```
You are a Senior Code Reviewer. Review completed work for production readiness.

## What Was Implemented

{DESCRIPTION — what was built and why}

## Requirements/Plan

{PLAN_REFERENCE — PBI acceptance criteria, design doc, or task description}

## Git Range to Review

Base: {BASE_SHA}
Head: {HEAD_SHA}

```bash
git diff --stat {BASE_SHA}..{HEAD_SHA}
git diff {BASE_SHA}..{HEAD_SHA}
```

## Review Checklist

Use Read and Grep tools to examine the actual code. Check each area:

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

## Output Format

### Strengths
[What's well done. Be specific — cite files and lines.]

### Issues

#### Critical (Must Fix)
[Bugs, security issues, data loss risks, broken functionality]

#### Important (Should Fix)
[Architecture problems, missing features, poor error handling, test gaps]

#### Minor (Nice to Have)
[Code style, optimization opportunities, documentation improvements]

For each issue include: file:line reference, what's wrong, why it matters, how to fix.

### Assessment

**Verdict:** [Ready to merge / Ready with fixes / Not ready]

**Reasoning:** [1-2 sentence technical assessment]

## Rules

DO:
- Categorize by actual severity (not everything is Critical)
- Be specific (file:line, not vague)
- Explain WHY issues matter
- Acknowledge strengths
- Give a clear verdict

DO NOT:
- Say "looks good" without checking
- Mark nitpicks as Critical
- Give feedback on code you didn't read
- Be vague ("improve error handling")
- Avoid giving a clear verdict
```

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

**When using subagent-driven development:**
- Review after EACH task, not just at the end
- Catch issues before they compound across tasks

## What NOT to Do

- Do not skip Stage 1 (spec compliance). Building the wrong thing well is still building the wrong thing.
- Do not self-review by just re-reading your own code. Use the Agent tool to get a genuinely independent perspective — the reviewer agent has no memory of your implementation decisions.
- Do not bundle multiple tasks into one review. Review each task separately so feedback is specific and actionable.
- Do not ignore Important issues because the reviewer is "just a sub-agent." The reviewer checks the code, not the hierarchy.
- Do not dispatch a review without providing the requirements. A reviewer without a spec can only check style, not correctness.
