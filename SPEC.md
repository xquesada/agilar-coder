# agilar-coder Specification

## Overview

Agilar AI SDLC CLI — project setup, skill management, and unattended PBI processing via Claude Code.

## Usage

```bash
agilar-coder init [directory]              # Set up a new project (scaffold wizard)
agilar-coder upgrade [directory]           # Update skills and docs to latest version
agilar-coder status [directory]            # Show installed version and skill status
agilar-coder run <backlog-file> [count]    # Process PBIs from a backlog
agilar-coder -h | --help                   # Show help
agilar-coder --version                     # Show version
```

## Self-Location

The script resolves its own repo root to find skills, scaffold, and VERSION:

1. Resolve `$0` via `readlink` (handles symlinks)
2. Check that the resolved directory contains `skills/` and `scaffold`
3. Fall back to `AGILAR_CODER_HOME` env var if step 2 fails

This allows the script to be symlinked into `$PATH` (e.g., `ln -s ~/projects/agilar-coder/agilar-coder /usr/local/bin/agilar-coder`).

## Version Tracking

- **Source of truth:** `VERSION` file at repo root (semver, e.g., `1.1.0`)
- **Project marker:** `.agilar-coder.version` in consumer project root (written by `init`, updated by `upgrade`)
- The `--version` flag and all subcommands read from the `VERSION` file

## Subcommands

### `init [directory]`

Set up a new project with the Agilar AI SDLC methodology.

**Arguments:**
- `directory` — (Optional) Target project directory. Defaults to `.`

**Behavior:**
1. Changes to the target directory
2. Runs the scaffold wizard (`scaffold` script from repo root)
3. Scaffold handles all interactive prompts (project name, stack, team mode, etc.)
4. After scaffold completes, writes `VERSION` to `.agilar-coder.version`

**Output:** Scaffold output + version confirmation

**Exit code:** `0` on success, `1` if scaffold not found or directory not found

### `status [directory]`

Show installed vs available version and skill inventory.

**Arguments:**
- `directory` — (Optional) Target project directory. Defaults to `.`

**Behavior:**
1. Changes to the target directory
2. Reads `.agilar-coder.version` (if present) for installed version
3. Reads `VERSION` from repo root for available version
4. Compares installed vs available skills (`implementations/claude-code/` vs `.claude/skills/`)
5. Reports missing skills (in repo but not installed) and extra skills (installed but not in repo)

**Output:**

```
agilar-coder status
═══════════════════
  Installed: 1.0.0
  Available: 1.1.0
  Status:    upgrade available

Skills:
  Installed: 17
  Available: 17
  Status:    all skills installed
```

If not initialized:

```
agilar-coder status
═══════════════════
  Installed: not initialized (run 'agilar-coder init')
  Available: 1.1.0
```

**Exit code:** `0`

### `upgrade [directory]`

Update installed skills and docs to the latest version.

**Arguments:**
- `directory` — (Optional) Target project directory. Defaults to `.`

**Behavior:**
1. Changes to the target directory
2. Requires `.agilar-coder.version` to exist (must be initialized first)
2. Copies updated skills from `implementations/claude-code/` → `.claude/skills/` (only changed files)
3. Detects skills in `.claude/skills/` that no longer exist in the repo; offers to remove them
4. Updates `docs/skills/` (canonical copies) if the directory exists
5. Notes CLAUDE.md template changes (does not overwrite — project CLAUDE.md is customized)
6. Updates `.agilar-coder.version` to current version

**Output:** List of added/updated/removed skills + version change summary

**Exit code:** `0` on success, `1` if not initialized

### `run <backlog-file> [count]`

Process PBIs from a backlog file. This is the explicit form of the default behavior.

**Arguments:**
- `backlog-file` — Path to markdown file containing the Product Backlog
- `count` — (Optional) Number of PBIs to process. If omitted, runs until backlog is empty.

All subcommands are explicit — passing a filename without `run` will error.

## Execution Modes (Run Mode)

| Mode | Trigger | Behavior |
|------|---------|----------|
| Bound | `count` provided | Process exactly `count` PBIs, then exit |
| Unbound | `count` omitted | Process PBIs until `REMAINING=0` |

## Help & Usage

**Triggers:** No arguments, `-h`, or `--help`

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
│ 0. Resolve repo root + VERSION          │
│    - Self-locate via $0 / symlink       │
│    - Fall back to AGILAR_CODER_HOME     │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 1. Check for subcommand / global flag   │
│    - init [dir] → cd dir, cmd_init      │
│    - upgrade [dir] → cd dir, cmd_upgrade│
│    - status [dir] → cd dir, cmd_status  │
│    - run → shift, fall through to run   │
│    - --version → print version, exit    │
│    - -h/--help → show help, exit        │
└──────────────────┬──────────────────────┘
                   ▼ (no subcommand matched → run mode)
┌─────────────────────────────────────────┐
│ 2. Parse run-mode flags + arguments     │
│    - --debug, -v/--verbose              │
│    - Validate backlog file exists       │
│    - Store count if provided            │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 3. Load config from ~/.agilar-coder/    │
│    - Create with defaults if missing    │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 4. Build prompt (fixed + user)          │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 5. If DEBUG=true:                       │
│    - Print config, command, prompt      │
│    - Exit 0 (do not invoke Claude)      │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 6. Validate project context             │
│    - Check for CLAUDE.md, code files,   │
│      project dirs, or manifest files    │
│    - If none found: exit 5              │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ 7. Run Claude Code                      │◄──────────┐
└──────────────────┬──────────────────────┘           │
                   ▼                                  │
┌─────────────────────────────────────────┐           │
│ 8. Check Claude exit code               │           │
│    - If non-zero: ABORT                 │           │
└──────────────────┬──────────────────────┘           │
                   ▼                                  │
┌─────────────────────────────────────────┐           │
│ 9. Read .agilar-status                  │           │
│    - If missing: ABORT                  │           │
│    - If STATUS=error: ABORT             │           │
└──────────────────┬──────────────────────┘           │
                   ▼                                  │
┌─────────────────────────────────────────┐           │
│ 10. Print progress                      │           │
│    "✓ Completed: <LAST_PBI>"            │           │
│    "  Remaining: 7/12"                  │           │
└──────────────────┬──────────────────────┘           │
                   ▼                                  │
┌─────────────────────────────────────────┐           │
│ 11. Check exit conditions               │           │
│     - REMAINING=0 → EXIT (done)         │           │
│     - Bound mode & iterations=count     │           │
│       → EXIT (quota reached)            │           │
│     - Otherwise → LOOP ─────────────────┼───────────┘
└─────────────────────────────────────────┘
```

## Output

**On start:**

```
agilar-coder v1.1.0
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
└── config.conf              # User configuration (run mode)

<project-directory>/
├── .agilar-coder.version    # Installed version marker (created by init)
├── .agilar-status           # Output: Status after each run (created by Claude)
├── .claude/skills/          # Installed skill implementations (created by init/upgrade)
├── docs/skills/             # Canonical skill references (created by scaffold)
├── product-backlog.md       # Input: Product Backlog (user-provided)
└── CLAUDE.md                # Project config (created by scaffold, customized by user)
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
