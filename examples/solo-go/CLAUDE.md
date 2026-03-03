# CLAUDE.md — chepibe-whatsapp

## Project

WhatsApp Business auto-responder for 4vQ bv. Monitors incoming messages and responds with business information, product availability, and order status.

## Team Mode

**Solo** — 1 human + 1 agent. Kanban. Trunk-based development.

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

Go 1.22+. Single binary. No framework — standard library HTTP + whatsmeow for WhatsApp Web protocol.

## Test Commands

```bash
go test ./...
```

## Lint Commands

```bash
go vet ./... && golangci-lint run ./...
```

## Build Commands

```bash
go build -o bin/chepibe-whatsapp ./cmd/chepibe-whatsapp
```

## Branching

Trunk-based. All commits on `main`. Run full test suite before every commit.

## PBI-First Rule

Any new work gets a PBI before starting. No exceptions. If the human forgets, the agent suggests it.

## Filesystem Backlog

PBI files live in `backlog/` and move through folders as they progress:

| Folder | Status | Contains |
|--------|--------|----------|
| `backlog/` | Backlog | Unrefined PBI files |
| `backlog/ready/` | Ready | Refined + planned PBIs, queued for execution |
| `backlog/in_progress/` | In Progress | PBIs currently being worked on |
| `backlog/done/` | Done | Completed PBIs (archive) |

**Execute ready PBIs:**
- "Build the next ready PBI" — picks oldest from ready/, executes, archives to done/
- "Build all ready PBIs" — processes the queue

**Sync:** `backlog.yaml` is the primary source of truth. The filesystem is a parallel record. Agent edits files first, then syncs. Inconsistencies trigger a warning.

## Product Backlog

PBIs are tracked in `backlog.yaml` in this repo.

**At session start:** read the backlog file, find PBIs with status `ready`. Read notes, acceptance criteria, and checklist before starting work. Update status to `in_progress` when you begin.

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


### Coaching Skills
- `scrum-master` — process conscience (always active)
- `po-coach` — product ownership guidance (during refinement)
- `facilitator` — Scrum event facilitation

## Architecture: Source and Instance

This repo is the **source** — it contains the code you build. It does not contain
runtime configuration or data.

The **instance** (where the software actually runs) lives at: /Users/4vq/projects/chepibe on M3

| What | Where |
|------|-------|
| Source code and tests | This repo |
| Configuration and settings | /Users/4vq/projects/chepibe on M3 |
| Data created at runtime | /Users/4vq/projects/chepibe on M3 |

When building, never add instance-specific configuration (secrets, server addresses,
user data) to this repo. When deploying, copy the built software to the instance
and let it read its configuration from there.

## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer (build and test here) |
| Production | M3 (4vq@100.91.174.29) |

Build the software here, then deploy the result to the production machine.
