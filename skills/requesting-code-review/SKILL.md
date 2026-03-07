# Requesting Code Review: When and How to Ask for Review

## Overview

A review request is only as useful as the context it provides. An incomplete request forces the reviewer to discover context you already have — wasting their time and producing a worse review.

**Core principle:** No review request without a complete dispatch. Give the reviewer everything they need upfront.

**Violating the letter of this process is violating the spirit of efficient collaboration.**

## Working Agreement

```
NO REVIEW REQUEST WITHOUT A COMPLETE DISPATCH
```

If the reviewer has to ask "what was this supposed to do?" or "where are the test results?", the dispatch was incomplete. Start over.

## When to Request Review

**Mandatory:**

- After completing a PBI or task — code review is part of the Definition of Done (see `SCRUM.md`)
- After implementing a plan's task — part of the subagent-driven workflow review cycle
- Before merging to `main` — in multi-agent and multi-human modes

**Optional but valuable:**

- After implementing a complex change — sanity check from a fresh perspective
- After fixing a subtle bug — second pair of eyes on the edge cases
- Before a major refactor — baseline check to ensure nothing is missed

## When NOT to Request Review

**Do not waste reviewer time on:**

- Work in progress (WIP) — finish it first, then request review
- Trivial changes (typo fixes, whitespace cleanup) — unless team policy requires it
- Code you have not tested — run the tests first, always
- Changes with no clear description — if you cannot describe what changed, you are not ready for review

Requesting review on untested code is disrespectful of the reviewer's time. They will find test failures you should have caught yourself.

## The Dispatch Template

Every review request must include the following. No exceptions, no shortcuts.

```
WHAT_WAS_IMPLEMENTED:
[1-2 sentence summary of what was built and why]

PLAN:
[The original requirements — PBI acceptance criteria, design doc section,
task description, or plan reference. Paste the full text, do not link
to something the reviewer cannot access.]

CHANGES:
- [file1]: [what changed and why]
- [file2]: [what changed and why]
- [file3]: [what changed and why]

BASE_SHA: [commit hash before changes started]
HEAD_SHA: [commit hash after all changes are committed]

TEST_RESULTS:
[Paste the actual test output. "Tests pass" is not evidence —
the output showing pass counts is evidence.]

NOTES:
[Anything the reviewer should know:
 - Trade-offs you made and why
 - Edge cases you considered
 - Open questions you want feedback on
 - Things you are unsure about
 - Scope decisions (what you intentionally did NOT build)]
```

### Why Each Field Matters

| Field | Purpose | Without It |
|-------|---------|------------|
| WHAT_WAS_IMPLEMENTED | Frames the review — reviewer knows what to expect | Reviewer wastes time figuring out the intent |
| PLAN | Enables spec compliance review (Stage 1) | Reviewer can only check code quality, not correctness |
| CHANGES | Guides attention to what matters | Reviewer reads everything equally, misses important parts |
| BASE_SHA / HEAD_SHA | Defines the exact diff to review | Reviewer reviews wrong range or stale code |
| TEST_RESULTS | Proves the code works before review | Reviewer finds test failures that should have been caught |
| NOTES | Shares context that is not in the code | Reviewer asks questions you could have preempted |

### Incomplete Dispatch Examples

**Bad dispatch:**
```
Can you review my changes? I implemented the auth feature.
```
Missing: plan, changes list, SHA range, test results, notes. The reviewer knows nothing.

**Bad dispatch:**
```
Review the diff between main and my branch. Tests pass.
```
Missing: plan, changes list, specific SHAs, actual test output, notes. "Tests pass" is a claim, not evidence.

**Good dispatch:**
```
WHAT_WAS_IMPLEMENTED:
Added JWT authentication middleware for the API. Users can now
log in and receive tokens that expire after 24 hours.

PLAN:
PBI #42 acceptance criteria:
- POST /auth/login accepts email + password, returns JWT
- JWT expires after 24h
- Protected endpoints return 401 without valid token
- Token refresh endpoint at POST /auth/refresh

CHANGES:
- pkg/auth/jwt.go: New — JWT generation and validation logic
- pkg/auth/middleware.go: New — HTTP middleware that extracts and validates tokens
- pkg/auth/handler.go: New — Login and refresh handlers
- cmd/api/routes.go: Added auth routes and middleware registration
- pkg/auth/jwt_test.go: Unit tests for token generation, validation, expiry
- pkg/auth/middleware_test.go: Integration tests for middleware chain
- pkg/auth/handler_test.go: Handler tests with mock store

BASE_SHA: abc1234
HEAD_SHA: def5678

TEST_RESULTS:
$ go test ./pkg/auth/ -v
=== RUN   TestGenerateToken
--- PASS: TestGenerateToken (0.00s)
=== RUN   TestValidateToken
--- PASS: TestValidateToken (0.00s)
[... full output ...]
PASS
ok      example.com/project/pkg/auth    0.847s

$ go test ./...
ok      example.com/project/cmd/api     1.234s
ok      example.com/project/pkg/auth    0.847s
ok      example.com/project/pkg/store   0.523s

NOTES:
- Chose HMAC-SHA256 over RSA for simplicity (single service, no token sharing)
- Token refresh reuses the same secret — separate refresh secret is a future enhancement
- Rate limiting on /auth/login is not in scope for this PBI
```

## Severity-Based Response After Review

After receiving review results, act based on severity:

| Severity | Action | Timeline |
|----------|--------|----------|
| Critical | Fix immediately, re-request review | Before any other work |
| Important | Fix before proceeding | Before next task |
| Minor | Note for later, or fix if trivial | When convenient |

For Critical and Important issues, the fix-and-re-review cycle continues until the reviewer gives a clean verdict. Do not merge with outstanding Critical issues. Ever.

## Self-Review Before Requesting External Review

Before dispatching a review to anyone (human or agent), do a quick self-check:

```
PRE-DISPATCH CHECKLIST:

[ ] All changes are committed (no uncommitted work)
[ ] Tests pass (full suite, not just your new tests)
[ ] Linter passes (if the project has one)
[ ] Build succeeds (if applicable)
[ ] Dispatch template is complete (all 6 fields filled)
[ ] Test results are pasted (not summarized)
[ ] Plan/requirements are included (not "see the PBI")
```

If any item fails, fix it before requesting review. The reviewer's job is to find issues you could not see, not issues you did not look for.

## Red Flags

**Never:**

- Request review without test results — test first, review second
- Send an incomplete dispatch — missing fields waste reviewer time
- Request review on untested code — disrespectful of reviewer's time
- Request review on WIP — finish the work first
- Say "the reviewer can figure it out" — they should not have to
- Request review and then immediately make more changes — the review is against a moving target
- Skip the dispatch template because "it's a small change" — small changes still need context
- Paste a link to the PBI instead of the actual requirements — the reviewer should not need to navigate elsewhere

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "It's a small change, no need for full dispatch" | Small changes still need context. The template scales down gracefully. |
| "The reviewer can look at the diff" | The diff shows what changed, not why or what it should do |
| "Tests obviously pass" | Paste the output. "Obviously" is not evidence. |
| "They know the codebase" | Knowing the codebase is not the same as knowing your intent |
| "I'll add details if they ask" | They should not have to ask. Provide upfront. |
| "Review is just a formality" | If it is a formality, your process is broken, not this skill |
| "I don't have time for the full template" | You don't have time for a bad review that misses real issues |

## Connection to Other Skills

| Skill | Relationship |
|-------|-------------|
| `skills/code-review/` | Defines how to PERFORM a review. This skill defines how to REQUEST one. The dispatch template is the input to the review process. |
| `skills/receiving-code-review/` | Defines how to RESPOND to review feedback. Completes the lifecycle: request, perform, receive. |
| `skills/verification/` | Test results are part of the dispatch template. Verification evidence is mandatory before requesting review. |
| `skills/subagent-driven/` | Review dispatch happens after each worker task. The orchestrator requests review as part of the task completion cycle. |
| `skills/tdd/` | TDD produces the test evidence that goes into the dispatch. No TDD = no test results = incomplete dispatch. |
| `skills/finishing-a-development-branch/` | Push+PR option triggers this skill. The PR description follows the dispatch template. |

## Connection to SCRUM.md

Code review is one of three working agreement gates in the Definition of Done. This skill ensures the gate is entered correctly — with a complete dispatch that enables an effective review. A poorly requested review produces a poor review, which means the gate is checked but not actually passed.

| Team Mode | How Review is Requested |
|-----------|----------------------|
| Solo | Human reviews agent code — dispatch includes what was built and why |
| Multi-agent | Orchestrator dispatches reviewer agent with template — agent reviews independently |
| Multi-human | Developer creates PR with dispatch content in description — peer reviews |

## Connection to DEVOPS.md

The Review Gate (Gate 4) requires code review before merge. This skill provides the entry protocol for that gate. Without a proper dispatch, the review gate degrades to rubber-stamping — the reviewer lacks context to find real issues, so they check style and miss substance.

The dispatch template maps directly to the review inputs defined in `skills/code-review/`:
- WHAT_WAS_IMPLEMENTED + PLAN = "What was implemented" + "Requirements or plan"
- BASE_SHA + HEAD_SHA = "The code changes"
- TEST_RESULTS + NOTES = Supporting evidence for the reviewer

## The Bottom Line

**No review request without a complete dispatch.**

Give the reviewer everything they need. The 5 minutes you spend filling out the template saves 30 minutes of back-and-forth and produces a review that actually finds real issues.
