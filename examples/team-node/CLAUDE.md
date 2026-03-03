# CLAUDE.md — acme-dashboard

## Project

Internal analytics dashboard for Acme Corp. React frontend, Express API, PostgreSQL. Three developers, two-week sprints.

## Team Mode

**Multi-human** — PO + 3 developers. Full Scrum. Feature branches + PRs.

## Working Agreements

These are non-negotiable. The agent enforces them.

- **No production code without a failing test first** (TDD)
- **No fix without root cause investigation** (Debugging)
- **No completion claim without fresh verification evidence** (Verification)
- **No design without exploring alternatives** (Brainstorming)
- **No merge without code review** (Code Review)

## Definition of Ready

A PBI is ready when:
- Small enough for a single sprint
- Acceptance criteria written (specific, testable)
- Design approved
- No blockers
- Checklist populated (if multi-step)

## Definition of Done

A PBI is done when:
- Acceptance criteria met (verified, not assumed)
- Tests pass (TDD — written before implementation)
- Verified with evidence (fresh test output after final change)
- Code reviewed
- Docs updated (if user-facing changes)

## Tech Stack

React 18 + TypeScript frontend. Express + TypeScript API. PostgreSQL 16. Vitest for testing. ESLint + Prettier for formatting.

## Test Commands

```bash
# All tests
npm test

# Frontend only
npm test -- --testPathPattern="src/"

# API only
npm test -- --testPathPattern="api/"
```

## Lint Commands

```bash
npx eslint . --ext .ts,.tsx && npx tsc --noEmit && npx prettier --check .
```

## Build Commands

```bash
npm run build
```

## Branching

Feature branches. PRs required. CI must pass before merge. Code review mandatory.

## PR Workflow

1. Create feature branch from `main`: `feat/pbi-42-user-dashboard`
2. Push branch, open PR
3. CI runs: lint + type check + tests
4. Peer reviews using `skills/code-review/` two-stage process
5. All checks green + approved → merge to `main`

## PBI-First Rule

Any new work gets a PBI before starting. No exceptions. If the human forgets, the agent suggests it.

## Skills

The following skills are active. The agent follows them as executable processes, not guidelines.

### Engineering Skills
- `brainstorming` — explore alternatives before designing
- `tdd` — red-green-refactor, every time
- `sprint-planning` — break PBIs into executable tasks
- `executing-plans` — execute plans with checkpoints
- `debugging` — root cause before fix
- `code-review` — two-stage review (spec compliance + quality)
- `verification` — evidence before claims
- `bdd` — acceptance tests as executable specifications

### Coordination Skills
- `git-worktrees` — optional, for developer isolation

### Coaching Skills
- `scrum-master` — process conscience (always active)
- `po-coach` — product ownership guidance (during refinement)
- `facilitator` — Scrum event facilitation

## Sprint Cadence

- 2-week sprints
- Sprint Planning: Monday morning
- Daily Standup: 9:30 AM
- Sprint Review: Friday afternoon (second week)
- Retrospective: immediately after review

## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer |
| Production | Acme internal servers (staging.acme-dashboard.internal → dashboard.acme.com) |

To deploy, connect to the production machine and pull the latest code, build, and restart.
