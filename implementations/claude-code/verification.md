---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# Verification Before Completion — Claude Code Implementation

> Canonical skill: `skills/verification/SKILL.md`
>
> This file adds Claude Code-specific tooling and workflow. The working agreement, gate function, rationalizations, and red flags in the canonical skill apply without modification. Read that file first.

## Working Agreement

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this response, you cannot claim it passes.

## Claude Code Verification Protocol

You have the Bash tool. You have full shell access. There is no reason to guess, assume, or extrapolate. Run the command and read the output.

### The Gate Function (Claude Code)

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY the verification command
   - Tests: the project's test runner (npm test, go test, pytest, cargo test, etc.)
   - Build: the project's build command
   - Lint: the project's linter
   - Runtime: the start command or a curl/request to verify behavior

2. RUN it with Bash tool
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
- Output from a previous Bash invocation before you made changes
- Partial test runs (running one test file when the claim is about the full suite)
- Linter output when claiming build success
- Agent sub-task reports without independent verification
- Moving a PBI file to `backlog/done/` without updating the backlog tool status

## Common Verification Sequences

### After Implementing a Feature (TDD)

```bash
# 1. Run the specific tests for the feature
npm test -- --testPathPattern="feature.test"
# or: go test ./pkg/feature/ -v
# or: pytest tests/test_feature.py -v

# 2. Run the full test suite
npm test
# or: go test ./...
# or: pytest

# 3. Run the linter if the project has one
npm run lint
# or: golangci-lint run
# or: ruff check .

# 4. Run the build if applicable
npm run build
# or: go build ./...
# or: cargo build
```

Report each result with the actual output.

### After Fixing a Bug

```bash
# 1. Run the reproduction test — must PASS now
npm test -- --testPathPattern="bug-repro.test"

# 2. Run the full suite — no regressions
npm test

# 3. Optionally: verify the fix manually
curl http://localhost:3000/endpoint-that-was-broken
```

### After Refactoring

```bash
# 1. Run the full test suite — nothing broke
npm test

# 2. Run the linter — no new issues
npm run lint

# 3. Run the build — still compiles
npm run build
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

## Red Flags in Claude Code Context

Stop and verify if you catch yourself:

- About to write "all tests pass" without a Bash tool invocation above it in the same response
- About to write "the build succeeds" without having run the build command
- Saying "this should fix it" without running the reproduction test
- Moving to the next task without running the full suite
- Using words like "should", "probably", "likely" about test or build state
- Copying output from an earlier Bash invocation that predates your latest code change

## Stale Evidence

Evidence becomes stale the moment you change code. If you:
1. Ran tests (all pass)
2. Made another edit (even a small one)
3. Want to claim "all tests pass"

You MUST run the tests again. The previous run is no longer evidence.

## Agent Delegation Verification

When delegating to sub-agents (Task tool), do not trust the sub-agent's completion claim. After the sub-agent returns:

1. Check the VCS diff to see what actually changed
2. Run the test suite yourself
3. Verify the changes match the requirements
4. THEN report the result to the human partner

## Self-Check Protocol

Before claiming ANY positive outcome in your response, scan your own response for trigger words:

### Trigger Words — STOP and Verify
- "should" — Did you actually run the command?
- "probably" — Replace with certainty from evidence
- "seems to" — Replace with observed output
- "looks good" — What specifically looks good? Cite evidence.
- "that fixes it" — Run the reproduction test
- "all tests pass" — Is there a Bash output above this in your response?
- "done" / "complete" — Have you verified against acceptance criteria?
- "moving on" — Did you run the full suite since your last change?

### The Protocol
1. Write your response
2. Before sending, scan for trigger words above
3. If any found: add verification evidence (Bash output) or rewrite the claim
4. Only send when every positive claim has evidence in the same response

## Acceptance Criteria as Verification Target

When verifying a PBI against its acceptance criteria, read the criteria from the backlog source — not from memory or conversation context. Use the po-coach skill's Backlog Access adapter to fetch the current PBI details and extract the acceptance criteria. This ensures verification checks against what was actually agreed, not what was discussed or paraphrased during the session.

For each acceptance criterion:
1. Read it from the backlog source (REST API response, YAML file, GitHub Issue body)
2. Identify the verification command or evidence needed
3. Run the verification
4. Report: criterion text → evidence → pass/fail

## Definition of Done Reference

Verification is part of the Definition of Done (`SCRUM.md`). The Verification Gate (`DEVOPS.md` Gate 3) requires fresh evidence before any PBI can be marked done. This implementation makes that gate executable: the Bash tool provides the evidence, the output in the response proves it was checked.
