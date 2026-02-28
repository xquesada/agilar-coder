---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---

# Test-Driven Development (TDD) — Claude Code Implementation

> Canonical skill: `skills/tdd/SKILL.md`
>
> This file adds Claude Code-specific tooling and workflow. The iron laws, rationalizations, and red flags in the canonical skill apply without modification. Read that file first.

## The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Write code before the test? Delete it. Start over. No exceptions.

## Claude Code TDD Workflow

### 1. RED - Write Failing Test

Use the Edit or Write tool to create the test file. One test, one behavior, clear name.

Then run it with Bash:

```bash
# Run only the new test — adapt command to the project's test runner
# Examples:
#   npm test -- --testPathPattern="path/to/test"
#   go test ./pkg/... -run TestRetryOperation
#   pytest path/to/test.py::test_retry_operation
#   cargo test retry_operation
```

**MANDATORY:** Execute the test command. Read the full output. Confirm:
- Test FAILS (not errors — compilation errors are not a failing test)
- Failure message matches expectation (missing function, wrong return value, etc.)
- Failure is because the feature doesn't exist yet

If the test passes immediately: you are testing existing behavior. Fix the test.

If the test errors (won't compile, import fails): fix the error, re-run until it fails correctly.

### 2. GREEN - Write Minimal Implementation

Use Edit to write the simplest code that makes the test pass. Nothing more.

Then run the test again:

```bash
# Same command as RED step — run the specific test
```

**MANDATORY:** Execute the test command. Read the full output. Confirm:
- The new test passes
- All other tests still pass
- No warnings or errors in output

If the test still fails: fix the implementation, not the test.

If other tests break: fix them now, before continuing.

### 3. REFACTOR - Clean Up (Tests Still Green)

Improve the code: remove duplication, improve names, extract helpers. Do not add behavior.

Run the full test suite after refactoring:

```bash
# Run the full suite — adapt to project
# Examples:
#   npm test
#   go test ./...
#   pytest
#   cargo test
```

**MANDATORY:** Confirm all tests still pass after refactoring.

### 4. Repeat

Next failing test for next behavior. Return to step 1.

## Verification Before Claiming Done

Before telling the human partner that work is complete, run the full test suite one final time and include the output in your response. Do not say "all tests pass" without showing the evidence.

See `implementations/claude-code/verification.md` for the full verification protocol.

Checklist (from canonical skill):
- [ ] Every new function/method has a test
- [ ] Watched each test fail before implementing (Bash output proves this)
- [ ] Each test failed for expected reason
- [ ] Wrote minimal code to pass each test
- [ ] All tests pass (Bash output proves this)
- [ ] Output pristine (no errors, warnings)
- [ ] Tests use real code (mocks only if unavoidable)
- [ ] Edge cases and errors covered

## Claude Code-Specific Guidance

### Use Bash, Not Assumptions

Always use the Bash tool to run tests. Never say "this should pass" — run it and read the output. Claude Code has full shell access; there is no reason to guess.

### Test Discovery

If you don't know the project's test runner, check:
- `package.json` for `scripts.test` (Node.js)
- `Makefile` or `go.mod` (Go)
- `pyproject.toml`, `setup.cfg`, or `pytest.ini` (Python)
- `Cargo.toml` (Rust)
- `.github/workflows/` for CI test commands

Use Grep or Read to find the correct command before running tests.

### One Test at a Time

Write one test. Run it. See it fail. Implement. Run it. See it pass. Then the next test.

Do not write multiple tests at once and then implement them all. That is batch-and-queue, not TDD.

### Mocking Discipline

Before mocking anything, ask:
1. What side effects does the real code have?
2. Does this test depend on any of those side effects?
3. Can I test with real code instead?

If you must mock, mock at the lowest level possible — the actual slow or external operation, not the high-level function the test depends on.

### Bug Fix Protocol

1. Write a test that reproduces the bug (RED)
2. Run it — confirm it fails for the right reason
3. Fix the bug (GREEN)
4. Run the test — confirm it passes
5. Run the full suite — confirm no regressions

Never fix a bug without a failing test first.

## Definition of Done Reference

TDD compliance is part of the Definition of Done (`SCRUM.md`). The Test Gate (`DEVOPS.md`) enforces that all tests pass before commit. Together, these ensure TDD is not optional — it is a pipeline requirement.
