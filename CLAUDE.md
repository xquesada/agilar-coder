# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project Overview

agilar-coder is Agilar's AI SDLC methodology made executable — a CLI, skills, scaffold wizard, and documentation in a single product. It encodes how humans and AI agents build software together: humans design, AI executes, quality is non-negotiable.

**Version:** See `VERSION` file (semver)

## Repository Structure

```
agilar-coder/
├── agilar-coder              # CLI bash script
├── SPEC.md                   # CLI specification
├── VERSION                   # Semver version (major.minor.patch)
├── README.md                 # Methodology overview + getting started
├── SCRUM.md                  # Agile framework: roles, artifacts, events
├── DEVOPS.md                 # Pipeline: environments, quality gates, CI/CD
├── TOOLCHAIN.md              # Recommended tools + alternatives
├── scaffold                  # Project setup wizard (bash)
├── skills/                   # 17 canonical skill definitions (tool-agnostic)
├── implementations/          # Tool-specific skill implementations
│   ├── claude-code/          # Claude Code .claude/skills/ format
│   ├── cursor/               # Cursor format (future)
│   └── generic/              # Plain markdown
├── templates/                # Scaffold templates (CLAUDE.md.tmpl, stacks/, agents/)
├── examples/                 # Example project configs (solo-go, team-node, multi-agent-web)
├── docs/                     # Case studies and implementation plans
└── .claude/
    └── settings.local.json   # Claude Code permissions
```

## The CLI (`agilar-coder`)

Single entry point for the Agilar AI SDLC: project setup, skill management, and unattended PBI processing.

```bash
./agilar-coder init                  # Set up a new project (scaffold wizard)
./agilar-coder upgrade               # Update skills and docs to latest version
./agilar-coder status                # Show installed version and skill status
./agilar-coder backlog.md            # Process all PBIs until done
./agilar-coder backlog.md 3          # Process exactly 3 PBIs
./agilar-coder --debug backlog.md    # Show command without executing
./agilar-coder --version             # Show version
```

### Self-Location

The script resolves its repo root from `$0` (follows symlinks), verifying `skills/` and `scaffold` exist. Falls back to `AGILAR_CODER_HOME` env var. This allows symlinking into `$PATH`.

### Version Tracking

- `VERSION` file at repo root — source of truth (semver)
- `.agilar-coder.version` in consumer projects — written by `init`, updated by `upgrade`

### Architecture

Single bash script, 14 functions:

- `resolve_repo_root()` — self-locate the agilar-coder repo via `$0` or env var
- `cmd_init()` — run scaffold wizard, write `.agilar-coder.version`
- `cmd_status()` — show installed/available version, compare skill inventories
- `cmd_upgrade()` — update skills, detect removals, bump version marker
- `load_config()` / `create_default_config()` — manage `~/.agilar-coder/config.conf`
- `validate_project_context()` — ensure working directory has code/CLAUDE.md
- `build_prompt()` — assemble fixed prompt + user prompt + backlog path
- `run_claude()` — invoke `claude` CLI with configured flags
- `read_status_file()` — parse `.agilar-status` after each run
- `main()` — subcommand dispatch + run-mode loop

### Communication Protocol

Claude writes `.agilar-status` with `TOTAL`, `REMAINING`, `LAST_PBI`, `STATUS` (and `ERROR` if failed). The script sources this file to determine next action.

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Backlog file not found / bad arguments |
| 2 | Claude Code failed |
| 3 | Status file missing/invalid |
| 4 | Status file reports error |
| 5 | No project context |

Full specification: `SPEC.md`

## The Methodology

Three Pillars:

1. **Human Designs, AI Executes** — clear role contract (human = PO, AI = skilled developer)
2. **Working Agreements** — quality gates the team commits to (TDD, debugging root cause, verification evidence, brainstorming alternatives, code review)
3. **Opinionated by Default** — concrete choices, not guidelines. Adapt when needed, but defaults work.

Details in: `SCRUM.md` (Agile framework), `DEVOPS.md` (pipeline + quality gates), `TOOLCHAIN.md` (recommended tools)

## Skills

17 skills covering the full development lifecycle:

| Category | Skills |
|----------|--------|
| **Planning** | brainstorming, sprint-planning, executing-plans |
| **Development** | tdd, debugging, bdd |
| **Quality** | code-review, requesting-code-review, receiving-code-review, verification |
| **Completion** | finishing-a-development-branch |
| **Multi-agent** | subagent-driven, parallel-agents, git-worktrees |
| **Facilitation** | scrum-master, po-coach, facilitator |

### Two layers

- **Canonical** (`skills/<name>/SKILL.md`) — tool-agnostic process definition
- **Implementations** (`implementations/<tool>/<name>.md`) — tool-specific format

When editing a skill, update the canonical SKILL.md first, then sync implementations. The canonical version is the source of truth.

## Scaffold

Interactive bash wizard that generates project configuration from templates.

```bash
./scaffold    # Run in your project directory
```

Asks about: project name, description, team mode (solo/multi-agent/multi-human), stack, tool, code review, branching, deployment, architecture, Entire audit trail.

**Testing scaffold changes:**
1. `bash -n scaffold` — syntax check
2. Run in a temp directory — verify generated files
3. Check that optional sections (deploy, architecture) appear/omit correctly

## Testing

- **CLI:** `shellcheck agilar-coder` for static analysis, `--debug` mode for prompt inspection
- **Scaffold:** `bash -n scaffold` + manual runs in temp directories
- **Skills:** Manual review — check that canonical and implementations stay in sync
- No automated test framework (bash scripts + markdown docs)

## Key Design Decisions

- **One PBI per execution** — prevents runaway processes, enables granular control
- **Project context required** — must have CLAUDE.md or existing code to prevent arbitrary tech choices
- **Status file interface** — shell-sourceable format for easy parsing
- **Skills are tool-agnostic** — canonical definitions separate from tool-specific implementations
- **Scaffold is opinionated** — generates working defaults, not blank templates
- **Engine + instance** — DEVOPS.md defines the Source/Instance architecture pattern

## Versioning

`VERSION` file at repo root. Semver convention:
- **Major** — breaking changes (skill renames, removed skills, changed scaffold output)
- **Minor** — new skills, new features
- **Patch** — content fixes, documentation updates
