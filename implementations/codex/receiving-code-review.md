---
name: receiving-code-review
description: Use when you receive code review feedback from a human, another agent, or CI — before making any changes in response to the review. Guides you through verifying claims, evaluating suggestions, and responding with evidence.
---

# Receiving Code Review — Codex Implementation

> Canonical skill: `skills/receiving-code-review/SKILL.md`
>
> This file adds Codex-specific workflow. The working agreement, 4-phase process, pushback protocol, and red flags in the canonical skill apply without modification. Read that file first.

## Working Agreement

```
NO PERFORMATIVE AGREEMENT — EVALUATE EVERY SUGGESTION INDEPENDENTLY
```

If you have not run a command or read the referenced code to verify a reviewer's claim, you cannot accept or reject it.

## Codex Review Response Protocol

You have full terminal access and can read any file. There is no reason to trust a reviewer's claim without checking. Verify everything before acting.

### The 4-Phase Protocol

```
BEFORE changing ANY code in response to review:

Phase 1: UNDERSTAND
1. Read the FULL review output
2. List each issue with its severity (Critical / Important / Minor)
3. Identify vague feedback that needs clarification

Phase 2: VERIFY (for each issue)
4. Read the file the reviewer references — confirm it exists and the line is correct
5. Run any test the reviewer claims should fail
6. Search for patterns the reviewer claims are missing
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

For each type of reviewer claim, use the appropriate verification:

| Reviewer Claim | How to Verify |
|---------------|---------------|
| "This test will fail" | Run the specific test in terminal and check |
| "This file is missing X" | Read the file and check for X |
| "This pattern is not followed" | Search for the pattern across the codebase |
| "This function has a bug" | Write and run a test that exercises the case |
| "This import is unused" | Search for usages of the import |
| "This endpoint is not tested" | Search test files for the endpoint |
| "Build will break" | Run the build command in terminal |

```bash
# Example: Reviewer claims "the error handling in handler.go is missing"
# VERIFY before accepting:

# 1. Read the actual code
cat handler.go

# 2. Check if error handling exists
grep -n "if err != nil" handler.go
# or: grep -n "catch" handler.js

# 3. If reviewer is right -> Accept and fix
# 4. If reviewer is wrong -> Push back with evidence (the actual code)
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
2. Process Critical issues first (verify -> evaluate -> fix/pushback)
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

# Run the build — fixes did not break the build
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

## What NOT to Do

- Do not agree with everything the reviewer says. Evaluate each suggestion independently — that is the entire point of this skill.
- Do not fix issues without verifying they are real. Read the code and run commands to check before changing anything.
- Do not ignore review feedback. Every item gets a response: accept, push back, or clarify.
- Do not make unrelated changes while fixing review issues. Fix what was found, nothing more. Unrelated changes in a review-fix commit make the next review harder.
- Do not skip post-fix verification. After fixing accepted issues, run the full test suite and paste the output.
- Do not get defensive. Present evidence, not emotions. "I disagree because [test output]" is professional. "The reviewer does not understand" is not.
- Do not treat all review sources equally. Human reviewers get full process. CI gets immediate compliance.
