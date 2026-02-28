---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging — Claude Code Implementation

This is the Claude Code implementation of the debugging skill. The canonical process definition is in `skills/debugging/SKILL.md`. Read it first — this file adds Claude Code-specific execution details, not a separate process.

## How to Execute Each Phase

### Phase 1: Root Cause Investigation

Use these tools to gather evidence before proposing any fix:

- **Read** error messages, stack traces, and log output completely. Do not skim.
- **Grep** for the error message, error code, or failing function name across the codebase to find related code and prior fixes.
- **Bash** to reproduce the issue. Run the failing test, trigger the bug, capture the output. If it doesn't reproduce, gather more data — don't guess.
- **Bash** `git diff` and `git log` to check recent changes that could have introduced the issue.
- **Read** every file in the call chain from the error back to the entry point. Trace the data flow backward.

**For multi-component systems**, add diagnostic logging at each component boundary before proposing a fix:

```bash
# Example: add temporary debug output at each layer
# Run once, read the output, identify which layer breaks
# THEN investigate that specific layer
```

Use **Agent** tool to parallelize evidence gathering in large codebases (e.g., "search for all callers of function X and check what values they pass for parameter Y").

**Hard gate.** Do not propose any fix until you can state: "The root cause is X because evidence Y shows Z."

### Phase 2: Pattern Analysis

- **Grep/Glob** to find working examples of similar code in the codebase.
- **Read** reference implementations completely — not just the parts that seem relevant.
- Compare working vs. broken code line by line. List every difference.
- **Read** dependency documentation, configuration files, and environment setup that the broken code relies on.

### Phase 3: Hypothesis and Testing

- State your hypothesis explicitly in conversation: "I believe the root cause is X because Y."
- Make the SMALLEST possible change to test the hypothesis. One variable only.
- **Bash** to run tests or reproduce after the minimal change.
- If it didn't work, state: "Hypothesis X was wrong because Z. New hypothesis: ..." Return to Phase 1 evidence if needed.
- Do NOT stack fixes. Revert the failed attempt before trying the next hypothesis.

### Phase 4: Implementation

1. **Write a failing test first.** Use the TDD skill (`skills/tdd/`) to write a test that demonstrates the bug. Run it to confirm it fails for the right reason.

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
- Do not use **Agent** tool to "go fix this bug" by delegation. Debugging requires understanding, and understanding requires you to trace the evidence yourself. Use Agent for evidence gathering, not for fix attempts.
- Do not revert to a working state and "try a different approach" without understanding why the current approach failed. That is not debugging — that is giving up.

## Integration with DEVOPS.md Root Cause Gate

This skill implements the Root Cause Gate from DEVOPS.md. The gate applies at any stage:

- Bug found during development: debug before fixing
- Test failure in CI: debug before patching
- Bug reported in production: debug before hotfixing

The temptation to skip the process is strongest when urgency is highest. That is precisely when the process matters most. Systematic debugging is faster than thrashing — always.
