---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs. Run verification commands and confirm output before making any success claims. Evidence before assertions, always.
---

# Verification Before Completion — Codex Implementation

> Canonical skill: `skills/verification/SKILL.md`
>
> This file adds Codex-specific workflow. The working agreement, gate function, rationalizations, and red flags in the canonical skill apply without modification. Read that file first.

## Working Agreement

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this response, you cannot claim it passes.

## Verification Protocol

You have terminal access. There is no reason to guess, assume, or extrapolate. Run the command and read the output.

### The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY the verification command
   - Tests: the project's test runner (npm test, go test, pytest, cargo test, etc.)
   - Build: the project's build command
   - Lint: the project's linter
   - Runtime: the start command or a curl/request to verify behavior

2. RUN it in terminal
   - Execute the FULL command, not a subset
   - Do not add flags that skip checks or reduce output
   - Do not pipe through head or tail — read everything

3. READ the complete output
   - Check the exit code
   - Count test results (passed, failed, skipped)
   - Look for warnings and errors, not just the summary line
   - If output is truncated, run again with output to a file and read the file

4. CONFIRM the output supports the claim
   - If NO: report the actual state with evidence (paste the relevant output)
   - If YES: make the claim WITH evidence (paste the relevant output)

5. ONLY THEN: tell the human partner the result
```

### What Counts as Evidence

| Claim | Evidence Required (paste in response) |
|-------|--------------------------------------|
| "All tests pass" | Test runner output showing pass count and 0 failures |
| "Build succeeds" | Build command output with exit code 0 |
| "Linter clean" | Linter output showing 0 errors, 0 warnings |
| "Bug is fixed" | Test output showing the reproduction test passes |
| "No regressions" | Full test suite output showing all pass |
| "Feature works" | Command output or test output demonstrating the behavior |
| "PBI is done" | Backlog tool API response showing status `done` |

### What Does NOT Count as Evidence

- "I looked at the code and it should work"
- Output from a previous terminal run before you made changes
- Partial test runs (running one test file when the claim is about the full suite)
- Linter output when claiming build success
- Delegated work reports without independent verification
- Moving a PBI file to `backlog/done/` without updating the backlog tool status

## Common Verification Sequences

### After Implementing a Feature (TDD)

```bash
# 1. Run the specific tests for the feature
# 2. Run the full test suite
# 3. Run the linter if the project has one
# 4. Run the build if applicable
```

Report each result with the actual output.

### After Fixing a Bug

```bash
# 1. Run the reproduction test — must PASS now
# 2. Run the full suite — no regressions
# 3. Optionally: verify the fix manually
```

### Before Committing

```bash
# Run everything the CI would run
npm test && npm run lint && npm run build
```

If any step fails, do not commit. Fix the issue first.

## Before Claiming PBI Done

This sequence must be followed in order:

1. Run the full test suite — capture output, confirm all pass
2. Verify each acceptance criterion with evidence
3. Update the backlog tool status to `done` (e.g., `curl -X PATCH .../api/backlog/:id -d '{"status":"done"}'`)
4. Only then move the PBI file to `backlog/done/`

If step 3 fails (tool unreachable), do NOT move the file. Add `## API Sync Needed` to the PBI file and tell the human partner.

## Stale Evidence

Evidence becomes stale the moment you change code. If you:
1. Ran tests (all pass)
2. Made another edit (even a small one)
3. Want to claim "all tests pass"

You MUST run the tests again. The previous run is no longer evidence.

## Self-Check Protocol

Before claiming ANY positive outcome in your response, scan your own response for trigger words:

### Trigger Words — STOP and Verify
- "should" — Did you actually run the command?
- "probably" — Replace with certainty from evidence
- "seems to" — Replace with observed output
- "looks good" — What specifically looks good? Cite evidence.
- "that fixes it" — Run the reproduction test
- "all tests pass" — Is there terminal output above this in your response?
- "done" / "complete" — Have you verified against acceptance criteria?
- "moving on" — Did you run the full suite since your last change?

### The Protocol
1. Write your response
2. Before sending, scan for trigger words above
3. If any found: add verification evidence (terminal output) or rewrite the claim
4. Only send when every positive claim has evidence in the same response

## Acceptance Criteria as Verification Target

When verifying a PBI against its acceptance criteria, read the criteria from the backlog source — not from memory or conversation context. This ensures verification checks against what was actually agreed, not what was discussed or paraphrased during the session.

For each acceptance criterion:
1. Read it from the backlog source (PBI file, REST API response, GitHub Issue body)
2. Identify the verification command or evidence needed
3. Run the verification
4. Report: criterion text -> evidence -> pass/fail

## Definition of Done Reference

Verification is part of the Definition of Done (`SCRUM.md`). The Verification Gate (`DEVOPS.md` Gate 3) requires fresh evidence before any PBI can be marked done. This implementation makes that gate executable: the terminal provides the evidence, the output in the response proves it was checked.
