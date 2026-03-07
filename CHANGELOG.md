# Changelog

All notable changes to agilar-coder are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/). Versions follow [Semantic Versioning](https://semver.org/).

Breaking changes include a **Migration** section with required manual steps.

## [1.5.2] - 2026-03-07

### Fixed
- `upgrade` silently aborted before writing `.agilar-coder.version` when no skills changed — arithmetic `((0 += 0))` returned exit code 1 under `set -e`
- `upgrade` silently aborted during CLAUDE.md section scan when `grep -v` filtered all lines — bare assignment exposed exit code 1 to `set -e`
- Plans written to `docs/plans/` instead of PBI files — template and generated content now explicitly say "write as `## Plan` inside the PBI file, never in `docs/plans/`"

## [1.5.1] - 2026-03-07

### Fixed
- `upgrade` CLAUDE.md prompt: skip sections whose generated content is only TODO stubs, preventing the same prompt from reappearing on every upgrade run

## [1.5.0] - 2026-03-07

### Fixed
- sprint-planning skill: removed `docs/plans/` references — plans go in PBI files as `## Plan` section (canonical + claude-code implementation)
- requesting-code-review skill: updated examples to reference PBI files instead of `docs/plans/` (claude-code + codex)
- git-worktrees skill: fixed front-matter `name: using-git-worktrees` → `name: git-worktrees` (claude-code + codex)
- bdd skill: expanded from 68-line stub to full implementation covering Gherkin examples, Scenario Outline, Background, tags, common mistakes, red flags, verification checklist (claude-code + codex)

### Added
- Smart CLAUDE.md section management: `ensure_section` handles missing headers, empty TODO stubs, and content under different headers
- Content generation for all 15 CLAUDE.md sections — auto-detects stack, team mode, and code review settings
- TODO stub detection: `section_is_empty` identifies sections with only `<!-- TODO -->` placeholders
- Scaffold backup: existing CLAUDE.md is saved to CLAUDE.md.bak before overwriting

### Changed
- `fill_gaps` now covers all 15 methodology sections (was 6), populates with real content instead of skipping
- `upgrade` CLAUDE.md handling: batched prompt with 3 options (add all / choose individually / skip) instead of per-section y/N prompts
- `upgrade` default changed from No to "add all with real content" — user ran upgrade, so adding content is the expected action
- `upgrade` individual mode shows 3-line content preview and defaults to Yes instead of No
- `upgrade` detects and offers to populate empty TODO sections (not just missing ones)
- `fill_gaps` no longer creates duplicate sections when a header exists but has empty content

## [1.4.1] - 2026-03-07

### Changed
- Named the methodology explicitly: "Agilar Agentic Coding methodology" throughout README, CLAUDE.md, templates, and agent definitions
- Equated "methodology", "method", and "framework" as synonyms in all entry points
- Added core principles and 9-step development lifecycle inline in CLAUDE.md template (no skill loading needed)
- Added PBI creation workflow to PBI-First Rule section in scaffold output (API-first for external backlogs, filesystem for local)

## [1.4.0] - 2026-03-07

### Added
- OpenAI Codex support: 18 skill implementations under `implementations/codex/`
- Multi-tool CLI: `install`, `upgrade`, `status`, and `assess` auto-detect Claude Code and Codex
- Tool detection functions: `detect_tools()`, `detect_tools_from_project()`
- Codex skills install to `~/.codex/skills/<name>/SKILL.md` (folder format)
- Tier 2 skills adapted for Codex (subagent-driven, parallel-agents, git-worktrees, using-agilar-coder)
- PBI creation workflow rule: API-first, API ID = local file name

## [1.3.4] - 2026-03-07

### Added
- Version management as core methodology: scaffold and fill-gaps generate VERSION + CHANGELOG.md
- Versioning section in CLAUDE.md template (semver convention, version impact classification)
- CHANGELOG update added to Definition of Done (SCRUM.md + CLAUDE.md template)
- Version impact classification in brainstorming skill (major/minor/patch during design)
- Version bump suggestion at PBI completion in using-agilar-coder skill (human confirms)
- assess checks for VERSION + CHANGELOG in Framework Tracking dimension
- fill-gaps detects and adds missing Product Backlog section in CLAUDE.md

## [1.3.3] - 2026-03-07

### Added
- CHANGELOG.md with version history and migration notes format
- `upgrade` command shows what changed between versions (CHANGELOG diff)
- `upgrade` command highlights manual migration steps for breaking changes
- `upgrade` detects missing CLAUDE.md template sections, explains each, and offers to append
- `fill-gaps` detects missing Product Backlog section in CLAUDE.md and prompts for backlog tool
- Upgrade workflow documented in README

### Fixed
- Strip `\r` from version files — prevents garbled upgrade output and failed version matching
- CLAUDE.md upgrade message replaced vague template review with actionable per-section prompts

## [1.3.2] - 2026-03-07

### Added
- CHANGELOG.md with retroactive entries from 1.0.0 through 1.3.1
- `upgrade` command shows what changed between installed and current version (from CHANGELOG)
- `upgrade` command highlights manual migration steps for breaking changes
- Upgrade workflow documented in README

### Fixed
- Strip `\r` from version files — prevents garbled upgrade output
- CLAUDE.md upgrade message now explains each missing section and offers to append it (replaces vague template review message)

## [1.3.1] - 2026-03-07

### Fixed
- Escape JSON quotes in SessionStart hook registration
- Only install Entire hook when Entire is enabled in the repo
- Gitignore instance artifacts generated by install/upgrade

## [1.3.0] - 2026-03-07

### Added
- `using-agilar-coder` skill — session bootstrapper / orchestrator with 9-step happy path (Capture, Refine, Plan, Implement, Review, Verify, Finish, Deploy, Close)
- Shortcut paths for bug fixes, config changes, research, pre-planned work
- Brainstorming Phase 4a: Design Concerns Scan (10 concerns checklist: Architecture, Data model, API, UI/UX, Security, Performance, Scalability, Observability, Edge cases, Migration/Rollback)
- NFR checkpoint in design — explicit statement per relevant non-functional requirement
- Compatible backlog tools concept in SCRUM.md — tools that are agilar-coder aware and proactively sync
- Sync workflow: check tool, create in tool, create file, reference file from tool

### Changed
- **BREAKING:** Design now lives in PBI file (`## Design` section), not in separate `docs/plans/` files
- **BREAKING:** `docs/decisions/` renamed to `docs/architecture/` for ADRs
- Brainstorming Phase 4 split into 4a (concerns scan) and 4b (elaborate design)
- Brainstorming Phase 5 writes design into PBI file, not to separate file
- PBI file format updated — living document that grows: Description, Design, AC, Plan, Notes
- SCRUM.md sync protocol rewritten with clear two-source model (tool = index, file = full document)
- SCRUM.md PBI file lifecycle updated with tool-first steps

### Migration
1. Rename `docs/decisions/` to `docs/architecture/` (if it exists)
2. Move any per-PBI designs from `docs/plans/` into their corresponding PBI files under a `## Design` section (if applicable)

## [1.2.0] - 2026-03-06

### Added
- `config` subcommand — view and edit `~/.agilar-coder/config.conf`
- SessionStart hook — shows startup message when Claude Code launches in an agilar-coder project
- Entire hook wrapper — shortens startup message for Entire-enabled projects
- Filesystem backlog section appended to CLAUDE.md on upgrade
- `fill_gaps()` creates PBIs for remaining gaps instead of leaving homework

### Fixed
- Fill-gaps now reaches 96%+ compliance; Entire creates PBI instead of hanging
- Handle reversed symlinks and split independent quality gate checks
- Register startup hook on 100% projects and fix idempotency message
- Strip trailing newline from version file in startup hook
- Use systemMessage JSON format for startup hook

## [1.1.0] - 2026-03-05

### Added
- `install` subcommand — smart install with consulting mode (detects existing methodology, offers fill-gaps or full scaffold)
- `upgrade` subcommand — update skills, detect removals, bump version marker
- `status` subcommand — show installed vs available version, compare skill inventories
- `assess` subcommand — 8-dimension methodology compliance scoring with color-coded progress bars
- Scaffold auto-detection: project name, tech stack, description detected automatically
- Modified skill detection — confirm before overwriting customized skills on upgrade
- Color-coded skills count in status output (green/yellow/red)

### Changed
- Renamed `init` to `install` (init kept as undocumented synonym)
- Scaffold UX overhaul: separate flows for existing vs greenfield projects
- Removed deployment and architecture questions from scaffold (auto-detected instead)
- Default code review to disabled for non-technical PO persona
- Auto-enable Entire; default audit trail based on team mode
- Contextual closing message for existing vs greenfield projects

## [1.0.0] - 2026-03-04

### Added
- CLI (`agilar-coder`) with `run` mode — unattended PBI processing via Claude Code
- Bound and unbound execution modes (process N PBIs or all)
- Status file interface (`.agilar-status`) for inter-process communication
- Debug mode (`--debug`) — inspect prompt without executing
- Spinner with timer and `--verbose` flag
- 17 skills covering planning, development, quality, completion, multi-agent, and facilitation
- Scaffold wizard — interactive project setup generating CLAUDE.md, skills, and configuration
- SCRUM.md — Agile framework with roles, artifacts, events
- DEVOPS.md — pipeline, environments, quality gates, Source/Instance architecture
- TOOLCHAIN.md — recommended tools and alternatives
- Multi-platform agent support (AGENTS.md, GEMINI.md symlinks)
- Entire audit trail scaffold step
- Backlog adapter pattern in po-coach skill
- Three examples: solo-go, team-node, multi-agent-web
- Case study: chepibe-core Sprint Zero
