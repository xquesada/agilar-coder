# Code Review

## Overview

Code review has two jobs: verify the implementation matches the spec, and verify the code is well-built. Both are mandatory. Neither substitutes for the other.

**Core principle:** Review early, review often. Code review is part of the Definition of Done (see SCRUM.md DoD). No merge without code review.

**Violating the letter of this process is violating the spirit of code review.**

**Optional for non-technical solo users:** If you're a non-technical PO working in solo mode and cannot review code meaningfully, this working agreement can be disabled during scaffold onboarding. When disabled, the agent relies on TDD, verification, automated quality checks (linting, warnings-as-errors, type checking), and BDD acceptance tests as the quality safety net. For multi-human teams, code review is always mandatory.

## Working Agreement

```
NO MERGE WITHOUT CODE REVIEW
```

This is the review gate defined in DEVOPS.md. Code that skips review does not advance to `main`. When this working agreement is disabled (non-technical solo mode), the other quality gates remain in effect.

## Two-Stage Review Process

Every review has two stages, in order. Do not skip stage 1 even if you're eager to discuss code quality.

### Stage 1: Spec Compliance

**Does the implementation match the requirements? Nothing more, nothing less.**

Verify:

- **Missing requirements** — Did the developer implement everything that was requested? Are there requirements they skipped or missed? Did they claim something works but didn't actually implement it?
- **Extra/unneeded work** — Did they build things that weren't requested? Over-engineer? Add "nice to haves" that weren't in spec?
- **Misunderstandings** — Did they interpret requirements differently than intended? Solve the wrong problem? Implement the right feature the wrong way?

**How to verify:** Read the actual code and compare it to the requirements line by line. Do not trust a developer's self-reported summary — verify independently.

Spec compliance must pass before proceeding to stage 2. If the implementation doesn't match the requirements, code quality is irrelevant — the wrong thing was built.

### Stage 2: Code Quality

**Is the implementation well-built?**

#### Code Quality Checklist

- Clean separation of concerns?
- Proper error handling?
- Type safety (if applicable)?
- DRY principle followed?
- Edge cases handled?
- Naming conventions clear and consistent?

#### Architecture Checklist

- Sound design decisions?
- Scalability considerations?
- Performance implications?
- Security concerns?
- Proper separation of concerns and loose coupling?

#### Testing Checklist

- Tests actually test logic (not mocks testing mocks)?
- Edge cases covered?
- Integration tests where needed?
- All tests passing?
- TDD discipline followed (tests written before production code)?

#### Production Readiness Checklist

- Migration strategy (if schema changes)?
- Backward compatibility considered?
- Documentation complete?
- No obvious bugs?
- Breaking changes documented?

## When to Request Review

**Mandatory:**
- After completing each task or PBI
- After implementing a major feature
- Before merge to `main`

**Optional but valuable:**
- When stuck (fresh perspective from another agent or human)
- Before refactoring (baseline check)
- After fixing a complex bug

## Performing a Review

### Input

A review requires:
- **What was implemented** — description of the work
- **Requirements or plan** — what it should do (PBI acceptance criteria, design doc, or task description)
- **The code changes** — diff, changed files, or commit range

### Output

Every review produces:

#### Strengths
What's well done. Be specific — cite files and line numbers.

#### Issues

Categorized by severity:

**Critical (Must Fix)**
Bugs, security issues, data loss risks, broken functionality. These block merge.

**Important (Should Fix)**
Architecture problems, missing features, poor error handling, test gaps. These should be fixed before proceeding.

**Minor (Nice to Have)**
Code style, optimization opportunities, documentation improvements. Note for later.

**For each issue, include:**
- File and line reference
- What's wrong
- Why it matters
- How to fix (if not obvious)

#### Assessment

Three verdicts:

| Verdict | Meaning |
|---------|---------|
| **Ready to merge** | No Critical or Important issues. Ship it. |
| **Ready with fixes** | Important issues exist but are straightforward. Fix them and merge — no re-review needed. |
| **Not ready** | Critical issues or fundamental problems. Fix and request re-review. |

Include a 1-2 sentence reasoning for the verdict.

## Acting on Review Feedback

After receiving a review:

1. **Fix Critical issues immediately.** These block everything.
2. **Fix Important issues before proceeding.** Do not defer them.
3. **Note Minor issues for later.** Address them when convenient, not urgently.
4. **Push back if the reviewer is wrong.** With technical reasoning — show code or tests that prove your point. Reviewers are not infallible.

## Red Flags

**Never:**
- Skip review because "it's simple"
- Ignore Critical issues
- Proceed with unfixed Important issues
- Rubber-stamp a review ("looks good" without actually reading the code)
- Mark nitpicks as Critical — severity must reflect actual impact
- Give feedback on code you didn't review

**If the reviewer is wrong:**
- Push back with technical reasoning
- Show code or tests that prove it works
- Request clarification on vague feedback

## Connection to Definition of Done

Code review is one of three working agreement gates in the DoD (see SCRUM.md):

| Working Agreement | DoD Gate | Skill |
|----------|----------|-------|
| No production code without a failing test first | Tests pass (TDD) | `skills/tdd/` |
| No completion claim without fresh verification evidence | Verified with evidence | `skills/verification/` |
| No merge without code review | Code reviewed | `skills/code-review/` (this skill) |

A PBI is not done until it passes code review. The review gate applies regardless of team mode:

| Team Mode | Reviewer |
|-----------|----------|
| Solo | Self-review (human verifies agent-written code before commit) |
| Multi-agent | Orchestrator reviews agent output |
| Multi-human | Peer review via PR |

## Connection to Other Skills

| Skill | Relationship |
|-------|-------------|
| `skills/requesting-code-review/` | Defines how to REQUEST a review — the dispatch template, when to request, what to include |
| `skills/receiving-code-review/` | Defines how to RESPOND to review feedback — the 4-phase process, pushback protocol, YAGNI check |
| `skills/verification/` | Verification evidence is required before requesting review — test output proves the code works |
| `skills/tdd/` | TDD discipline is checked during quality review (Stage 2) — tests before code |
| `skills/subagent-driven/` | The two-stage review process from this skill is used for every worker task |

Together, these three skills form a complete review lifecycle: **request** (`requesting-code-review`) → **perform** (`code-review`, this skill) → **receive** (`receiving-code-review`).

## Real-World Impact

From code review sessions:
- Spec compliance catches (stage 1): 30% of reviews find missing or misunderstood requirements
- Code quality catches (stage 2): 60% of reviews find at least one Important issue
- Post-review bug rate: significantly lower than unreviewed code
- Two-stage process prevents the most expensive bug: building the wrong thing
