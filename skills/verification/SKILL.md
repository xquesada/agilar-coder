# Verification Before Completion

## Overview

Claiming work is complete without verification is dishonesty, not efficiency.

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## Working Agreement

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command since the last change, you cannot claim it passes.

## Enforcement

Verification is not a suggestion. It is enforced through two mechanisms:

1. **Definition of Done** (see `SCRUM.md`) — A PBI is not done unless verification evidence exists. Saying "it works" is not evidence. Test output, build output, or observed behavior is evidence.

2. **Verification Gate** (see `DEVOPS.md` Gate 3) — No PBI is marked done without fresh evidence generated after the final change. Stale evidence from a previous run does not count.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command or action proves this claim?
2. EXECUTE: Run the FULL verification (fresh, complete)
3. READ: Full output, check exit code, count failures
4. CONFIRM: Does output support the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit or create a PR without verification
- Trusting agent success reports without independent check
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**

## Red Flags — Structured Detection

| Signal | Risk | Action |
|--------|------|--------|
| Using "should", "probably", "seems to" | Claiming without evidence | STOP — run the verification command |
| Expressing satisfaction ("Great!", "Perfect!") | Premature completion claim | STOP — have you run tests since last change? |
| About to commit or create PR | Shipping unverified code | STOP — run full test suite + lint + build |
| Trusting agent success report | Agent may have hallucinated results | STOP — verify independently with Bash |
| Relying on partial test run | Regressions in untested areas | STOP — run the FULL suite, not a subset |
| "Just this once" thinking | Normalizing unverified claims | STOP — no exceptions, ever |
| Copying output from earlier run | Evidence is stale after code change | STOP — re-run after every code change |
| Moving to next task without verifying current | Incomplete PBI marked done | STOP — verify before advancing |

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence is not evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter is not compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion is not an excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

## Before/After: Rationalization in Practice

### Before (WRONG)
Developer implements a fix, runs linter (passes), says "Build passes, moving on."
Reality: Linter is not compiler. Build was never run. The fix may not compile.

### After (CORRECT)
Developer implements a fix, runs linter (passes), runs build command (exit 0), runs tests (34/34 pass), says "All checks pass: linter clean, build succeeds, 34 tests green."

### Before (WRONG)
Agent sub-task returns "Task complete, all tests pass."
Orchestrator says "Great, task 3 is done, moving to task 4."
Reality: Agent's claim was never verified independently.

### After (CORRECT)
Agent sub-task returns "Task complete, all tests pass."
Orchestrator checks VCS diff, runs test suite independently, confirms 34/34 pass.
Orchestrator says "Task 3 verified: diff reviewed, 34 tests pass independently."

## Key Patterns

### Tests

```
CORRECT:  Run test command -> See "34/34 pass" -> "All tests pass"
WRONG:    "Should pass now" / "Looks correct"
```

### Regression Tests (TDD Red-Green)

```
CORRECT:  Write test -> Run (pass) -> Revert fix -> Run (MUST FAIL) -> Restore -> Run (pass)
WRONG:    "I've written a regression test" (without red-green verification)
```

### Build

```
CORRECT:  Run build command -> See exit 0 -> "Build passes"
WRONG:    "Linter passed" (linter doesn't check compilation)
```

### Requirements

```
CORRECT:  Re-read plan -> Create checklist -> Verify each item -> Report gaps or completion
WRONG:    "Tests pass, phase complete"
```

### Agent Delegation

```
CORRECT:  Agent reports success -> Check VCS diff -> Verify changes -> Report actual state
WRONG:    Trust agent report at face value
```

## Why This Matters

Unverified completion claims cause:
- Broken trust with the human partner — "I don't believe you" is a relationship failure
- Undefined functions shipped — code that crashes on first use
- Missing requirements shipped — incomplete features delivered as complete
- Time wasted on false completion, redirect, rework — the most expensive kind of waste
- Violation of honesty as a core value

A single false completion claim costs more than every verification step combined.

## When To Apply

**ALWAYS before:**
- ANY variation of success or completion claims
- ANY expression of satisfaction about work state
- ANY positive statement about code correctness
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents
- Reporting status to the human partner

**Rule applies to:**
- Exact phrases ("all tests pass", "it works", "done")
- Paraphrases and synonyms ("looks good", "should be fine", "that fixes it")
- Implications of success ("moving on to the next item")
- ANY communication suggesting completion or correctness

## The Bottom Line

**No shortcuts for verification.**

Run the command. Read the output. THEN claim the result.

This is non-negotiable.
