# PBI #289: Multi-tool CLI support (Codex + Claude Code)

**Epic:** agilar-ai-sdlc
**Status:** Backlog

## Description

The CLI currently hardcodes `implementations/claude-code/` as the only skill source and `.claude/skills/` as the only install target. Refactor `install`, `upgrade`, `status`, and `assess` to support multiple tools.

### What changes

1. **Tool detection** — On `install`/`upgrade`, detect which tools are present in the consumer project:
   - Claude Code: `.claude/` directory or `claude` CLI available
   - Codex: `codex` CLI available or Codex project markers (TBD based on Codex conventions)
   - Install skills for all detected tools (both can coexist in the same project)

2. **Skill installation paths**:
   - Claude Code: `implementations/claude-code/*.md` → `.claude/skills/*.md` (existing)
   - Codex: `implementations/codex/*.md` → Codex skill target directory (TBD — needs research into where Codex expects skills)

3. **Functions to update**:
   - `fill_gaps()` — currently loops over `implementations/claude-code/` only
   - `run_full_scaffold()` — conflict detection hardcoded to `.claude/skills/`
   - `cmd_status()` — skill count hardcoded to `implementations/claude-code/` and `.claude/skills/`
   - `cmd_upgrade()` — same
   - `assess_skills_library()` — same
   - `check_claude_cli()` — should become `check_tool_cli()` or similar, checking for whichever tools are detected

4. **No new flags needed** — auto-detect is the default. A `--tool` flag could be added later if users want to force a specific tool.

## Acceptance Criteria

- [ ] `agilar-coder install` detects available tools and installs skills for all of them
- [ ] `agilar-coder upgrade` upgrades skills for all detected tools
- [ ] `agilar-coder status` shows skill counts per tool
- [ ] `agilar-coder assess` checks skills for all detected tools
- [ ] Projects with only Claude Code work exactly as before (backwards compatible)
- [ ] Projects with only Codex get Codex skills installed
- [ ] Projects with both get both sets of skills installed

## Dependencies

- Requires PBI #290 (Codex skill implementations exist in `implementations/codex/`)
- Needs research: where does Codex expect skill files in a consumer project?

## Notes

- Keep the refactor minimal — extract the tool-specific paths into variables/functions, don't over-abstract
- The `run` subcommand (unattended PBI execution) is Claude Code-specific for now; Codex equivalent can be a separate PBI
