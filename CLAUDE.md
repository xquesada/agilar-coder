# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

agilar-coder is a bash script that runs Claude Code unattended, processing Product Backlog Items (PBIs) from a markdown file. It orchestrates Claude Code to implement one PBI at a time, tracking progress via a status file.

## Running the Script

```bash
# Process all PBIs until backlog is empty
./agilar-coder backlog.md

# Process exactly N PBIs
./agilar-coder backlog.md 3

# Show full command without executing (debug mode)
./agilar-coder --debug backlog.md
```

## Testing Changes

No automated test framework. To verify changes:
1. Use `--debug` mode to inspect prompt construction without execution
2. Run with `shellcheck agilar-coder` for static analysis
3. Test with a sample backlog file in a project directory

## Architecture

Single 413-line bash script with 10 functions:

**Configuration & Setup:**
- `load_config()` / `create_default_config()` - Manage `~/.agilar-coder/config.conf`
- `validate_project_context()` - Ensure working directory has code/CLAUDE.md

**Prompt Engineering:**
- `build_prompt()` - Assembles fixed prompt + user prompt + backlog path
- `FIXED_PROMPT` constant - Core instructions for PBI processing

**Execution:**
- `run_claude()` - Invokes `claude` CLI with configured flags
- `read_status_file()` - Parses `.agilar-status` after each run
- `main()` - Loop: run Claude → check status → print progress → repeat or exit

**Communication Protocol:**
Claude writes `.agilar-status` with `TOTAL`, `REMAINING`, `LAST_PBI`, `STATUS` (and `ERROR` if failed). The script sources this file to determine next action.

## Key Design Decisions

- **One PBI per execution**: Prevents runaway processes; enables granular control
- **Project context required**: Must have CLAUDE.md or existing code to prevent arbitrary tech choices
- **Status file interface**: Shell-sourceable format for easy parsing
- **Fixed + User prompt**: Core behavior is hardcoded; user customizes via `USER_PROMPT` config

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Backlog file not found |
| 2 | Claude Code failed |
| 3 | Status file missing/invalid |
| 4 | Status file reports error |
| 5 | No project context |
