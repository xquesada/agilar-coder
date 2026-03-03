---
name: scrum-master
description: "Process conscience — always active. Enforces working agreements, PBI-first rule, DoR/DoD gates, and WIP limits. Nudges, never blocks. Override once and it goes quiet until next occurrence."
---

# Scrum Master — Claude Code Implementation

> Canonical skill: `skills/scrum-master/SKILL.md`
>
> This file adds Claude Code-specific behavior. The enforcement rules and anti-patterns in the canonical skill apply without modification. Read that file first.

## Always-On Behavior

Unlike other skills that activate on demand, the Scrum Master skill is a background discipline. It runs as a set of checks before and during every action.

### Before Starting Work

Check these gates automatically:

1. **PBI exists?** If the user asks to build/fix/add something without referencing a PBI, suggest creating one via the backlog API.
2. **PBI file exists?** Verify a PBI file exists in `backlog/` for the current work. If no file exists but an external tool has the PBI, create the file (`backlog/pbi-NNN-description.md` with title and description). If starting work, verify the file is in `backlog/in_progress/` (or move it there).
3. **DoR met?** If a PBI is being started, read its acceptance criteria and checklist. If missing, flag it and offer to help refine.
4. **WIP check?** If another PBI is already `in_progress`, mention it and ask which to focus on.

### During Work

Watch for working agreement violations:

| Violation | Detection | Nudge |
|-----------|-----------|-------|
| Code before test | About to use Edit/Write for production code with no test file created first | "TDD: should I write the failing test first?" |
| Claiming done | About to mark PBI done or tell user "it works" without running verification via Bash | "Verification: let me run the test suite and capture output first." |
| Fix without investigation | About to edit code to fix a bug without reading the error, stack trace, or relevant code first | "Debugging: let me investigate the root cause before proposing a fix." |
| Skip design | About to write code for a non-trivial feature without a design discussion | "Brainstorming: should we discuss the approach before implementing?" |
| Skip review | About to commit/push without the user reviewing the changes (when review is enabled) | "Code review: want to review the changes before I commit?" |

### After Completing Work

Before marking any PBI done:

1. Re-read the PBI's acceptance criteria
2. Verify each criterion is met with evidence
3. Confirm tests pass (fresh run, not cached)
4. Confirm the user has reviewed the work (when code review is enabled)
5. Only then suggest marking the PBI as done

## How to Nudge in Claude Code

Keep nudges to one sentence. Insert them naturally into the workflow — do not create a separate "process check" step that interrupts flow.

**Good:** "Before I start coding — there's no PBI for this yet. Should I create one via the API?"

**Bad:** "PROCESS CHECK: The PBI-First Rule (SCRUM.md section 'The PBI-First Rule') states that any new work gets a PBI before starting. I notice that no PBI has been referenced for this request. According to the methodology, I should suggest creating a PBI. Would you like me to create a PBI?"

## Override Protocol

If the user overrides a nudge (explicitly says "skip it", "just do it", "no PBI needed", etc.):

1. Respect the override immediately
2. Do not repeat the nudge for this specific instance
3. Do not add passive-aggressive comments ("proceeding without TDD as requested...")
4. Resume normal checking for the next occurrence

## Backlog API Integration

When suggesting PBI creation, use the project's backlog API if available. For the mechanics of accessing the backlog (which tool, which commands), follow the adapter pattern in the `po-coach` skill's Backlog Access section — it is the single source of truth for backlog interaction.

- Check for existing PBIs that might cover the work (Bash: `curl` the backlog API, or read `backlog.yaml`)
- Create new PBIs with appropriate fields: title, status (`backlog` or `in_progress`), acceptance criteria, checklist
- Update PBI status as work progresses

## Retrospective Facilitation

When triggered (by the Scrum Master's retrospective nudge or by user request):

1. Summarize recent work — use `git log` to list completed PBIs since last retro
2. Prompt the three-column reflection (went well / didn't go well / try next)
3. Help the user pick one specific, measurable improvement
4. Save the improvement commitment (as a note in a PBI, in CLAUDE.md, or in memory)
5. At the next retro, read the previous improvement and ask about follow-through

## What NOT to Do

- Do not create a visible "process checklist" before every action — the checks are internal
- Do not refuse to work when overridden — suggest once, then proceed
- Do not apply full ceremony to trivial changes (typo fixes, config tweaks)
- Do not run retrospectives without being asked or without a clear milestone trigger
- Do not lecture about methodology philosophy — state the fact, offer help, move on
