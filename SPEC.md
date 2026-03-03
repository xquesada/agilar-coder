# agilar-coder Specification

## Overview

Agilar's opinionated way of doing AI-augmented Agile Software Development — project setup, skill management, and unattended PBI processing via Claude Code.

## Usage

```bash
agilar-coder install [git repository]           # Set up a new project (scaffold wizard or fill-gaps)
agilar-coder upgrade [git repository]           # Update the framework to the latest version
agilar-coder status [git repository]            # Show installed version of the framework
agilar-coder assess [git repository]            # Assess methodology compliance
agilar-coder run <backlog-file> [count]         # Build PBI's from a Product Backlog in unattended mode
agilar-coder -h | --help                        # Show help
agilar-coder --version                          # Show version
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

### `install [git repository]`

Set up a new project with the Agilar AI SDLC methodology. Detects existing methodology artifacts and offers consulting mode.

**Arguments:**
- `git repository` — (Optional) Target git repository directory. Defaults to `.`

**Behavior:**
1. Changes to the target directory
2. Checks for existing methodology signals (CLAUDE.md, .claude/skills/, Makefile, docs/, etc.)
3. If no signals found → **greenfield mode**: runs scaffold wizard directly
4. If signals found → **consulting mode**:
   a. Runs `assess` to show current compliance
   b. Offers three choices: fill gaps only, full install, or abort
   c. Fill-gaps mode: installs missing skills, creates AGENTS.md/GEMINI.md symlinks, backlog structure, docs/skills/, and .agilar-coder.version — without touching existing files
   d. Full install: runs scaffold wizard as before (may overwrite)
5. Writes `VERSION` to `.agilar-coder.version`

**Output:** Assessment report (consulting mode) or scaffold output (greenfield) + version confirmation

**Exit code:** `0` on success, `1` if scaffold not found or directory not found

### `status [git repository]`

Show installed version of the framework and skill inventory.

**Arguments:**
- `git repository` — (Optional) Target git repository directory. Defaults to `.`

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

### `upgrade [git repository]`

Update the framework to the latest version.

**Arguments:**
- `git repository` — (Optional) Target git repository directory. Defaults to `.`

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

### `assess [git repository]`

Assess project methodology compliance against Agilar AI SDLC standards.

**Arguments:**
- `git repository` — (Optional) Target git repository directory. Defaults to `.`

**Behavior:**
1. Changes to the target directory
2. Evaluates 8 methodology dimensions via file-existence checks and content grep
3. Prints color-coded compliance report with progress bars
4. Lists actionable recommendations for gaps

**Assessment Dimensions:**

| # | Dimension | What is checked | Scoring |
|---|-----------|----------------|---------|
| 1 | Agent Guidance | CLAUDE.md, AGENTS.md, GEMINI.md | 0-3 points |
| 2 | Working Agreements | TDD, code review, verification, debugging, brainstorming (grep in CLAUDE.md, docs/, skills/) | 0-5 points |
| 3 | Quality Gates | Test command, lint config, CI config, pre-commit hook | 0-4 points |
| 4 | Skills Library | Count of `.claude/skills/*.md` vs 17 available | ratio (0-17) |
| 5 | Documentation | README, DEVOPS.md/equivalent, stack reference | 0-3 points |
| 6 | CI/CD Pipeline | GitHub Actions, GitLab CI, Jenkins, CircleCI, Bitbucket | 0-1 (present/absent) |
| 7 | Backlog Management | backlog/ dir, backlog.yaml/md, GitHub Issues template | 0-1 (present/absent) |
| 8 | Framework Tracking | .agilar-coder.version exists and is current | 0-2 points |

**Color coding:** Green (80-100%), Yellow (40-79%), Red (0-39%)

**Output:**

```
agilar-coder assess
═══════════════════
Project: my-project

  Agent Guidance       ██████░░░░ 67%   CLAUDE.md ✓  AGENTS.md ✓  GEMINI.md ✗
  Working Agreements   ██████████100%   TDD ✓  Review ✓  Verification ✓  Debug ✓  Brainstorm ✓
  ...

  Overall: 52% Agilar-compliant

Recommendations:
  1. Install 15 missing skills               agilar-coder install
  ...
```

**Exit code:** `0` (informational command, always succeeds)

### `run <backlog-file> [count]`

Build PBI's from a Product Backlog in unattended mode.

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

## Claude Code Prerequisite

Before running any operational subcommand (`install`, `upgrade`, `status`, `assess`, `run`), the script checks that:

1. `claude` is in PATH (`command -v claude`)
2. `claude --version` succeeds (proves it's executable)

If either check fails, the script prints install instructions and exits with code 1. This check is skipped for `--help` and `--version`.

## Assumptions

- The Product Backlog format is defined by the user (Claude interprets it)
- Claude Code is installed and available in PATH as `claude` (enforced by prerequisite check)
- User has appropriate permissions in the working directory
- The backlog file uses some consistent convention for marking PBIs as DONE (Claude will follow whatever pattern exists or establish one)
