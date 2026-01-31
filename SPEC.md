# agilar-coder Specification

## Overview

A bash script that runs Claude Code unattended, processing Product Backlog Items (PBIs) from a markdown file.

## Usage

```bash
agilar-coder <backlog-file.md> [count]
agilar-coder -h | --help
agilar-coder --debug <backlog-file.md> [count]
```

- `backlog-file.md` — Path to markdown file containing the Product Backlog
- `count` — (Optional) Number of PBIs to process. If omitted, runs until backlog is empty.

## Execution Modes

| Mode | Trigger | Behavior |
|------|---------|----------|
| Bound | `count` provided | Process exactly `count` PBIs, then exit |
| Unbound | `count` omitted | Process PBIs until `REMAINING=0` |

## Help & Usage

**Triggers:** No arguments, `-h`, or `--help`

```bash
agilar-coder
agilar-coder -h
agilar-coder --help
```

**Output:**

```
agilar-coder - Run Claude Code unattended on a Product Backlog

USAGE:
    agilar-coder <backlog-file> [count]
    agilar-coder -h | --help
    agilar-coder --debug <backlog-file> [count]

ARGUMENTS:
    backlog-file    Path to markdown file containing the Product Backlog
    count           (Optional) Number of PBIs to process
                    If omitted, runs until backlog is empty

OPTIONS:
    -h, --help      Show this help message
    --debug         Show full command without executing

CONFIGURATION:
    ~/.agilar-coder/config.conf

    Available settings:
        USER_PROMPT   Custom instructions for Claude (git, CI, etc.)
        MODEL         Claude model to use (default: opus)
        MAX_TURNS     Max turns per PBI (default: 100)
        VERBOSE       Enable verbose output (default: false)

EXAMPLES:
    agilar-coder backlog.md          Process all PBIs until done
    agilar-coder backlog.md 3        Process exactly 3 PBIs
    agilar-coder ./docs/sprint.md    Use backlog from subdirectory

EXIT CODES:
    0    Success
    1    Backlog file not found
    2    Claude Code failed
    3    Status file missing or invalid
    4    Status file indicates error
    5    No project context (missing CLAUDE.md and no code files)
```

**Exit code for help:** `0` (showing help is not an error)

## Debug Mode

**Trigger:** `--debug` flag

```bash
agilar-coder --debug backlog.md
agilar-coder --debug backlog.md 3
```

**Behavior:**

- Parses arguments and loads config normally
- Builds the full prompt and command
- Prints the complete command that *would* be executed
- Does NOT invoke Claude Code
- Exits with code 0

**Output:**

```
agilar-coder --debug
═════════════════════════════════════════

CONFIG:
  File: ~/.agilar-coder/config.conf
  MODEL=opus
  MAX_TURNS=100
  VERBOSE=false
  USER_PROMPT="After completing each PBI, commit with message..."

BACKLOG:
  File: ./backlog.md
  Mode: bound (3 PBIs)

COMMAND:
  claude --print --dangerously-skip-permissions --model opus --max-turns 100 "<prompt>"

FULL PROMPT:
─────────────────────────────────────────
You are processing a Product Backlog. Your task:

1. Parse the provided backlog file and identify all PBIs...
[... full fixed prompt ...]

Additional instructions:
After completing each PBI, commit with message...

Backlog file to process: ./backlog.md
─────────────────────────────────────────

═════════════════════════════════════════
Debug mode: command was NOT executed.
```

## Configuration

**Location:** `~/.agilar-coder/config.conf`

**Format:** Shell-sourceable key=value pairs

```bash
# Custom prompt for development workflow (git, CI, code style, etc.)
USER_PROMPT="After completing each PBI, commit with message 'feat: <PBI title>'. Run tests before marking done."

# Claude model (default: opus)
MODEL=opus

# Max agentic turns per PBI (default: 100)
MAX_TURNS=100

# Verbose output (default: false)
VERBOSE=false
```

If config file doesn't exist, script creates it with defaults on first run.

## Prompt Structure

The prompt sent to Claude Code is constructed from two parts:

### Fixed Prompt (hardcoded in script)

```
You are processing a Product Backlog. Your task:

1. Parse the provided backlog file and identify all PBIs (Product Backlog Items)
2. Find the FIRST PBI that is NOT marked as DONE
3. Implement that ONE PBI completely
4. Update the backlog file to mark that PBI as DONE
5. Write a status file ".agilar-status" in the current directory with:
   TOTAL=<total number of PBIs>
   REMAINING=<number of PBIs not yet DONE>
   LAST_PBI=<identifier or title of the PBI you just completed>
   STATUS=<success|error>

If there are no PBIs, or all PBIs are already DONE, write:
   TOTAL=<count>
   REMAINING=0
   LAST_PBI=none
   STATUS=success

If the backlog is invalid or you encounter an error, write:
   STATUS=error
   ERROR=<description>

IMPORTANT: Work on exactly ONE PBI per execution. Do not continue to the next PBI.
```

### User Prompt (from config)

Appended after the fixed prompt. Contains user's custom instructions.

### Final Prompt Assembly

```
<fixed prompt>

Additional instructions:
<user prompt>

Backlog file to process: <backlog-file-path>
```

## Claude Code Invocation

**Fixed flags (always applied):**

- `--print` — Non-interactive mode
- `--dangerously-skip-permissions` — Skip all permission prompts

**Configurable flags:**

- `--model <MODEL>` — From config (default: opus)
- `--max-turns <MAX_TURNS>` — From config (default: 100)
- `--verbose` — If VERBOSE=true in config

**Example invocation:**

```bash
claude --print --dangerously-skip-permissions --model opus --max-turns 100 "<full-prompt>"
```

## Status File: `.agilar-status`

Written by Claude after each run in the current working directory.

```bash
TOTAL=12
REMAINING=7
LAST_PBI=PBI-005: Implement user login
STATUS=success
```

Or on error:

```bash
STATUS=error
ERROR=Could not parse backlog format
```

## Script Flow

```
┌─────────────────────────────────────────┐
│ 0. Check for flags                      │
│    - No args, -h, --help → show help    │
│    - --debug → set DEBUG=true           │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 1. Parse arguments                      │
│    - Validate backlog file exists       │
│    - Store count if provided            │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 2. Load config from ~/.agilar-coder/    │
│    - Create with defaults if missing    │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 3. Build prompt (fixed + user)          │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 4. If DEBUG=true:                       │
│    - Print config, command, prompt      │
│    - Exit 0 (do not invoke Claude)      │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 5. Validate project context             │
│    - Check for CLAUDE.md, code files,   │
│      project dirs, or manifest files    │
│    - If none found: exit 5              │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 6. Run Claude Code                      │◄──────────┐
└──────────────────┬──────────────────────┘           │
                   ▼                                  │
┌─────────────────────────────────────────┐           │
│ 7. Check Claude exit code               │           │
│    - If non-zero: ABORT                 │           │
└──────────────────┬──────────────────────┘           │
                   ▼                                  │
┌─────────────────────────────────────────┐           │
│ 8. Read .agilar-status                  │           │
│    - If missing: ABORT                  │           │
│    - If STATUS=error: ABORT             │           │
└──────────────────┬──────────────────────┘           │
                   ▼                                  │
┌─────────────────────────────────────────┐           │
│ 9. Print progress                       │           │
│    "✓ Completed: <LAST_PBI>"            │           │
│    "  Remaining: 7/12"                  │           │
└──────────────────┬──────────────────────┘           │
                   ▼                                  │
┌─────────────────────────────────────────┐           │
│ 10. Check exit conditions               │           │
│     - REMAINING=0 → EXIT (done)         │           │
│     - Bound mode & iterations=count     │           │
│       → EXIT (quota reached)            │           │
│     - Otherwise → LOOP ─────────────────┼───────────┘
└─────────────────────────────────────────┘
```

## Output

**On start:**

```
agilar-coder v1.0
Backlog: ./product-backlog.md
Mode: bound (3 PBIs)
─────────────────────────────
Starting iteration 1...
```

**After each PBI:**

```
✓ Completed: PBI-005: Implement user login
  Progress: 5/12 done, 7 remaining
─────────────────────────────
Starting iteration 2...
```

**On completion:**

```
─────────────────────────────
✓ agilar-coder complete
  Total PBIs processed: 3
  Backlog status: 7 PBIs remaining
```

**On error:**

```
✗ Error: Claude Code failed (exit code 1)
  Check output above for details.
  Aborting.
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success (completed requested work) |
| 1 | Backlog file not found |
| 2 | Claude Code failed |
| 3 | Status file missing or invalid |
| 4 | Status file indicates error |
| 5 | No project context found |

## File Structure

```
~/.agilar-coder/
└── config.conf          # User configuration

<project-directory>/
├── product-backlog.md   # Input: Product Backlog (user-provided)
├── .agilar-status       # Output: Status after each run (created by Claude)
└── CLAUDE.md            # Optional: Claude Code project config (read by Claude)
```

## Error Handling

| Condition | Action |
|-----------|--------|
| Backlog file doesn't exist | Print error, exit 1 |
| Config file missing | Create with defaults, continue |
| Claude Code exits non-zero | Print error, exit 2 |
| `.agilar-status` missing after run | Print error, exit 3 |
| `STATUS=error` in status file | Print ERROR message, exit 4 |
| No project context found | Print error with guidance, exit 5 |

## Project Context Requirement

Before running, the script validates that the working directory has sufficient context for Claude to work with. This prevents Claude from making arbitrary decisions about tech stack, architecture, etc.

**Valid context includes any of:**

- `CLAUDE.md` file (recommended for greenfield projects)
- Code files (`.py`, `.js`, `.ts`, `.go`, `.rs`, `.java`, `.rb`, etc.)
- Project structure directories (`src/`, `lib/`, `app/`, `pkg/`, etc.)
- Project manifest files (`package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, etc.)

**For greenfield projects**, create a `CLAUDE.md` file with tech stack information:

```markdown
# Project: MyApp

## Tech Stack
- Language: TypeScript
- Framework: Next.js 14 (App Router)
- Database: PostgreSQL with Prisma
- Testing: Vitest

## Conventions
- Use functional components
- API routes in /app/api
```

If no context is found, the script exits with code 5.

## Assumptions

- The Product Backlog format is defined by the user (Claude interprets it)
- Claude Code is installed and available in PATH as `claude`
- User has appropriate permissions in the working directory
- The backlog file uses some consistent convention for marking PBIs as DONE (Claude will follow whatever pattern exists or establish one)
