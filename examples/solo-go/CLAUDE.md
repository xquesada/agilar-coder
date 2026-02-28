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

## Skills

The following skills are active. The agent follows them as executable processes, not guidelines.

### Engineering Skills
- `brainstorming` — explore alternatives before designing
- `tdd` — red-green-refactor, every time
- `writing-plans` — plan before building
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

## Project-Specific Notes

- Service conventions: see `docs/plans/2026-02-28-service-conventions-design.md` in the chepibe repo
- Deployment: systemd unit on M3 (macOS launchd), auto-restart on failure
- WhatsApp session stored in `data/whatsapp.db` (SQLite, gitignored)
- Environment variables: `WHATSAPP_DB_PATH`, `RESPONSE_CONFIG_PATH`
