# Design: Add Entire Audit Trail Step to Scaffold Wizard

**Date:** 2026-03-01
**Status:** Approved

## Context

The agilar-coder TOOLCHAIN.md recommends Entire as a git-native audit trail for AI agent work (status: recommended, not required). However, the scaffold wizard never asks about it — the recommendation only exists in documentation that users may not read.

Entire is tool-agnostic (not Claude Code-specific). It works with any AI assistant.

## Design

### New Wizard Step

Insert as **step 8** (after code review, before project type). Simple Y/n prompt:

```
Audit trail:
  Entire captures a git-native log of every AI agent session (what was asked,
  what changed, what tests ran). Recommended for teams; optional for solo.
Enable Entire? [y/N]:
```

- Default: **No** — it's recommended-not-required, and beginners shouldn't be nudged into installing extra tooling
- Variable: `ENTIRE_ENABLED` (yes/no)

### CLAUDE.md Output

When enabled, generate an `## Audit Trail` section after the Branching section:

```markdown
## Audit Trail

**Entire** is enabled. Every AI agent session is captured as a git-native transcript:
what was asked, what the agent did, what files changed, what tests ran.

- Transcripts are stored in git history alongside the code changes
- The verification skill captures evidence automatically when Entire is active
- For retrospectives, use session transcripts to review what went well

Install: see [Entire documentation](https://github.com/entirejs/entire) for setup instructions.
```

When disabled: no section generated (same pattern as architecture/deploy).

### Summary Output

When enabled, add to "What was created":

```
  Entire                       — Git-native audit trail (install separately)
```

And in "Next steps", insert before "Create your first PBI":

```
  N. Install Entire for automatic session capture (see CLAUDE.md)
```

### Step Renumbering

Current steps 8-10 become steps 9-11. Total: 11 steps. All comments in the script update accordingly.

## Approach

Approach A (recommended): New standalone step between code review and project type. Both code review and Entire are about traceability — natural grouping. Clean separation from the "what are you building" questions that follow.

Rejected alternatives:
- **B: Bundle with code review** — two concepts in one step, less clear
- **C: Add at the end** — disconnected from related traceability questions
