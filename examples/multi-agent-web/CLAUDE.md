# CLAUDE.md — acme-dashboard

## Project

Internal analytics dashboard for Acme Corp. React frontend, Express API, PostgreSQL. One developer orchestrating multiple agents.

## Team Mode

**Multi-agent** — 1 human + agent team. Kanban. Worktrees from main.

## Working Agreements

These are non-negotiable. The agent enforces them.

- **No production code without a failing test first** (TDD)
- **No fix without root cause investigation** (Debugging)
- **No completion claim without fresh verification evidence** (Verification)
- **No design without exploring alternatives** (Brainstorming)
- **No merge without code review** (Code Review)

## Definition of Ready

A PBI is ready when:
- Small enough for a single working session
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

Worktrees from `main`. Each agent works in isolation. Self-merge after tests pass.

### Multi-Agent Worktree Workflow

**Orchestrator** (main session) coordinates. **Workers** (sub-agents) implement in isolated worktrees.

1. Orchestrator creates a branch per worker before dispatch
2. Worker implements in its worktree following TDD
3. Code reviewer agent verifies (spec compliance + quality)
4. Orchestrator merges to main, runs full test suite
5. Push to remote after all tests pass

**Port Assignments:**

| Agent | Dev Server | Test | Debug |
|-------|-----------|------|-------|
| Orchestrator | 3000 | 3100 | 3200 |
| Worker 1 | 3001 | 3101 | 3201 |
| Worker 2 | 3002 | 3102 | 3202 |
| Worker 3 | 3003 | 3103 | 3203 |
| Worker 4 | 3004 | 3104 | 3204 |

**Worker instructions:** Workers receive a focused prompt with task, context, constraints, and TDD requirement. They have no conversation history — include everything they need.

## PBI-First Rule

Any new work gets a PBI before starting. No exceptions. If the human forgets, the agent suggests it.

## Product Backlog

PBIs are tracked in `backlog.yaml` in this repo.

**At session start:** read the backlog file, find PBIs with status `ready`. Read notes, acceptance criteria, and checklist before starting work. Update status to `in_progress` when you begin.

## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer |
| Production | Acme internal servers (staging.acme-dashboard.internal → dashboard.acme.com) |

To deploy, connect to the production machine and pull the latest code, build, and restart.

## Skills

The following skills are active. The agent follows them as executable processes, not guidelines.

### Engineering Skills
- `brainstorming` — explore alternatives before designing
- `tdd` — red-green-refactor, every time
- `writing-plans` — plan before building
- `executing-plans` — execute plans with checkpoints
- `debugging` — root cause before fix
- `code-review` — two-stage review (spec compliance + quality)
- `requesting-code-review` — when and how to request review
- `receiving-code-review` — responding to review feedback
- `verification` — evidence before claims
- `bdd` — acceptance tests as executable specifications

### Coordination Skills
- `subagent-driven` — orchestrate sub-agents for independent tasks
- `parallel-agents` — dispatch parallel work
- `git-worktrees` — agent isolation via worktrees
- `finishing-a-development-branch` — clean branch completion

### Coaching Skills
- `scrum-master` — process conscience (always active)
- `po-coach` — product ownership guidance (during refinement)
- `facilitator` — Scrum event facilitation
