---
name: receiving-code-review
description: Use when you receive code review feedback from a human, another agent, or CI — before making any changes in response to the review
---

# Receiving Code Review — Claude Code Implementation

> Canonical skill: `skills/receiving-code-review/SKILL.md`
>
> This file adds Claude Code-specific tooling and workflow. The working agreement, 4-phase process, pushback protocol, and red flags in the canonical skill apply without modification. Read that file first.

## Working Agreement

```
NO PERFORMATIVE AGREEMENT — EVALUATE EVERY SUGGESTION INDEPENDENTLY
```

If you have not used the Bash, Read, or Grep tools to verify a reviewer's claim in this response, you cannot accept or reject it.

## Claude Code Review Response Protocol

You have full shell access, file reading, and search tools. There is no reason to trust a reviewer's claim without checking. Verify everything before acting.

### The 4-Phase Protocol (Claude Code)

```
BEFORE changing ANY code in response to review:

Phase 1: UNDERSTAND
1. Read the FULL review output (if from Agent tool, read the complete response)
2. List each issue with its severity (Critical / Important / Minor)
3. Identify vague feedback that needs clarification

Phase 2: VERIFY (for each issue)
4. Use Read to examine the code the reviewer references
5. Use Bash to run any test the reviewer claims should fail
6. Use Grep to verify claims about missing patterns or conventions
7. Record: verified / not reproduced / needs clarification

Phase 3: EVALUATE (for each verified issue)
8. Apply the YAGNI check from the canonical skill
9. Decide: Accept / Push Back / Clarify

Phase 4: RESPOND
10. Fix accepted issues (one at a time, with verification after each)
11. Document pushback with evidence
12. Ask for clarification on vague items

ONLY THEN: Report the response to the human partner
```

### Phase 2 Verification Commands

For each type of reviewer claim, use the appropriate tool:

| Reviewer Claim | Verification Tool | Command |
|---------------|-------------------|---------|
| "This test will fail" | Bash | Run the specific test and check |
| "This file is missing X" | Read | Read the file and check for X |
| "This pattern is not followed" | Grep | Search for the pattern across the codebase |
| "This function has a bug" | Bash | Write and run a test that exercises the case |
| "This import is unused" | Grep | Search for usages of the import |
| "This endpoint is not tested" | Grep | Search test files for the endpoint |
| "Build will break" | Bash | Run the build command |

```bash
# Example: Reviewer claims "the error handling in handler.go is missing"
# VERIFY before accepting:

# 1. Read the actual code
# Use Read tool: Read handler.go

# 2. Check if error handling exists
# Use Grep: search for error handling patterns in handler.go
# Pattern: "if err != nil" or "catch" or ".catch(" depending on language

# 3. If reviewer is right → Accept and fix
# 4. If reviewer is wrong → Push back with evidence (the actual code)
```

### Verifying Agent Reviewer Claims

Agent reviewers (dispatched via the Agent tool for spec compliance or code quality) can hallucinate findings. Extra caution required:

```bash
# Agent reviewer says: "Missing test for edge case in pkg/auth/jwt.go line 42"

# Step 1: Does that file and line even exist?
# Use Read tool to read pkg/auth/jwt.go around line 42

# Step 2: Does the claimed edge case matter?
# Use Bash to check test coverage:
go test ./pkg/auth/ -v -run TestJWT

# Step 3: If the edge case is real and untested → Accept
# Step 4: If the line doesn't exist or edge case is already covered → Push back
```

### Verifying CI Failures

CI failures are facts, not suggestions. Do not push back on CI.

```bash
# CI reports test failure — reproduce locally:
npm test        # or: go test ./... or: pytest
# Fix the failure. No discussion needed.

# CI reports lint failure — reproduce locally:
npm run lint    # or: golangci-lint run or: ruff check .
# Fix the violation. Lint rules are project decisions, not suggestions.

# CI reports build failure — reproduce locally:
npm run build   # or: go build ./... or: cargo build
# Fix the build. Broken builds block everything.
```

## Pushback Response Template

When pushing back on a reviewer's suggestion, use this structure:

```
PUSHBACK on [issue description]:

The reviewer observed: [what they said]

Verification: [what I checked and found]
- Ran: [command]
- Output: [relevant output]
- Read: [file:line] — shows [what it shows]

Assessment: The current implementation is correct because [reasoning].

Recommendation: Keep the current approach.
[OR: Consider alternative X instead, which addresses the concern without Y trade-off.]
```

## Handling Multiple Review Issues

When a review contains multiple issues, process them in severity order:

```
1. List all issues from the review
2. Process Critical issues first (verify → evaluate → fix/pushback)
3. Process Important issues second
4. Process Minor issues last (or note for later)
5. Run the FULL test suite after ALL fixes
6. Report: what was fixed, what was pushed back, what needs clarification
```

Do not fix one issue and immediately respond. Process the entire review, then respond once with all decisions.

### Post-Fix Verification

After fixing accepted issues:

```bash
# Run the full test suite — no regressions from fixes
npm test        # or: go test ./... or: pytest

# Run the linter — no new lint issues from fixes
npm run lint    # or: golangci-lint run or: ruff check .

# Run the build — fixes didn't break the build
npm run build   # or: go build ./... or: cargo build
```

Evidence of passing tests after fixes is part of the response. Fixes without verification evidence are unverified fixes (see `skills/verification/`).

## Clarification Request Template

When feedback is too vague to act on:

```
CLARIFICATION NEEDED on [issue description]:

The reviewer said: "[exact quote]"

I need specifics:
- Which part of the code is this about? (file and line)
- What specifically is wrong? (expected vs. actual)
- What would you change? (concrete suggestion)

Blocked: No changes will be made for this item until clarification arrives.
```

## Integration with Subagent-Driven Workflow

When the orchestrator receives review feedback from a reviewer agent:

1. Process the review using the 4-phase protocol above
2. For accepted fixes: dispatch a worker agent with specific fix instructions
3. For pushback: document the reasoning in the task log
4. For clarification: escalate to the human partner (vague agent feedback is usually a hallucination — verify first)
5. After worker fixes: run full verification before marking the task complete

## What NOT to Do

- Do not agree with everything the reviewer says. Evaluate each suggestion independently — that is the entire point of this skill.
- Do not fix issues without verifying they are real. Use Read, Grep, and Bash to check before changing code.
- Do not ignore review feedback. Every item gets a response: accept, push back, or clarify.
- Do not make unrelated changes while fixing review issues. Fix what was found, nothing more. Unrelated changes in a review-fix commit make the next review harder.
- Do not skip post-fix verification. After fixing accepted issues, run the full test suite and paste the output.
- Do not get defensive. Present evidence, not emotions. "I disagree because [test output]" is professional. "The reviewer doesn't understand" is not.
- Do not treat all review sources equally. Human reviewers get full process. Agent reviewers get extra verification. CI gets immediate compliance.
