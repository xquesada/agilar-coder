---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes. Follow the 4-phase process to find root cause first.
---

# Systematic Debugging — Codex Implementation

> Canonical skill: `skills/debugging/SKILL.md`
>
> This file adds Codex-specific workflow. The process, working agreements, and red flags in the canonical skill apply without modification. Read that file first.

## How to Execute Each Phase

### Phase 1: Root Cause Investigation

Gather evidence before proposing any fix:

- **Read** error messages, stack traces, and log output completely. Do not skim.
- **Search for** the error message, error code, or failing function name across the codebase to find related code and prior fixes.
- **Run in terminal** to reproduce the issue. Run the failing test, trigger the bug, capture the output. If it doesn't reproduce, gather more data — don't guess.
- **Run** `git diff` and `git log` to check recent changes that could have introduced the issue.
- **Read** every file in the call chain from the error back to the entry point. Trace the data flow backward.

**For multi-component systems**, add diagnostic logging at each component boundary before proposing a fix. Run once, read the output, identify which layer breaks, then investigate that specific layer.

**Hard gate.** Do not propose any fix until you can state: "The root cause is X because evidence Y shows Z."

### Phase 2: Pattern Analysis

- Search for and find files matching working examples of similar code in the codebase.
- Read reference implementations completely — not just the parts that seem relevant.
- Compare working vs. broken code line by line. List every difference.
- Read dependency documentation, configuration files, and environment setup that the broken code relies on.

### Phase 3: Hypothesis and Testing

- State your hypothesis explicitly: "I believe the root cause is X because Y."
- Make the SMALLEST possible change to test the hypothesis. One variable only.
- Run in terminal to run tests or reproduce after the minimal change.
- If it didn't work, state: "Hypothesis X was wrong because Z. New hypothesis: ..." Return to Phase 1 evidence if needed.
- Do NOT stack fixes. Revert the failed attempt before trying the next hypothesis.

### Phase 4: Implementation

1. **Write a failing test first.** Follow the TDD skill (`skills/tdd/`) to write a test that demonstrates the bug. Run it to confirm it fails for the right reason.

2. **Implement a single fix.** Address the root cause. ONE change. No "while I'm here" improvements.

3. **Run the full test suite:**
   ```bash
   # Run the specific failing test
   # Then run the full suite to check for regressions
   ```

4. **If 3+ fixes have failed**, STOP. Do not attempt fix number 4. Tell the user:
   - How many fixes you've attempted
   - What each one revealed
   - Why this suggests an architectural problem rather than a bug
   - Ask whether to continue debugging or rethink the approach

### After Fixing

Follow the verification skill (`skills/verification/`) to provide fresh evidence that the fix works:

- Test output showing the previously-failing test now passes
- Full test suite output showing no regressions
- If the bug was user-visible, demonstrate the correct behavior

## What NOT to Do

- Do not propose "try this and see if it works" without completing Phase 1. That is guess-and-check, not debugging.
- Do not make multiple changes at once. If the tests pass, you won't know which change fixed it.
- Do not skip the failing test in Phase 4. The test is how you prove the bug existed and is now fixed.
- Do not revert to a working state and "try a different approach" without understanding why the current approach failed. That is not debugging — that is giving up.

## Integration with DEVOPS.md Root Cause Gate

This skill implements the Root Cause Gate from DEVOPS.md. The gate applies at any stage:

- Bug found during development: debug before fixing
- Test failure in CI: debug before patching
- Bug reported in production: debug before hotfixing

The temptation to skip the process is strongest when urgency is highest. That is precisely when the process matters most. Systematic debugging is faster than thrashing — always.
