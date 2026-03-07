---
name: using-agilar-coder
description: "Session bootstrapper — invoke BEFORE any response or action. Establishes which skills to use and when. If a skill might apply, invoke it first."
---

# Using Agilar Coder — Claude Code Implementation

> Canonical skill: `skills/using-agilar-coder/SKILL.md`
>
> This file adds Claude Code-specific behavior. The skill inventory, priority rules, and red flags in the canonical skill apply without modification.

## How Skills Work in Claude Code

Skills are markdown files in `.claude/skills/`. They appear in the system prompt's skill list and are invoked via the **Skill** tool. When you invoke a skill, its content is loaded — follow it directly.

**Never use the Read tool on skill files.** Use the Skill tool.

## The Happy Path: Idea to Production

The canonical skill defines a 9-step default lifecycle. Here is how each step maps to Claude Code actions:

| Step | Skill | Claude Code Action |
|------|-------|--------------------|
| 1. Capture | scrum-master | Check backlog tool → create PBI in tool → create `backlog/pbi-NNN.md` → reference file from tool |
| 2. Refine | po-coach, brainstorming | Invoke skills. Add `## Design` + `## Acceptance Criteria` to PBI file |
| 3. Plan | sprint-planning | Invoke skill. Add `## Plan` to PBI file |
| 4. Implement | tdd, debugging, bdd | Invoke per task. Use executing-plans or subagent-driven for multi-task |
| 5. Review | requesting-code-review | Dispatch Agent tool reviewers (spec + quality) |
| 6. Verify | verification | Run test/lint/build via Bash, read full output |
| 7. Finish | finishing-a-development-branch | Merge, clean up worktrees, post-merge test |
| 8. Deploy | (project-specific) | Follow CLAUDE.md deploy workflow via Bash/SSH |
| 9. Close | scrum-master | Move PBI to `backlog/done/`, update CHANGELOG.md, suggest version bump, update docs/CLAUDE.md/memory |

**When a user says "build this"**, the default path is: Step 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9. Scale ceremony to complexity — a bug fix skips Steps 2-3, a config change skips Steps 5-7. But the default is the full path.

## Session Start Behavior

On every new message from the user, determine where in the happy path this request falls:

1. **New idea / feature / change?** → Start at Step 1 (capture) or Step 2 (refine) → invoke **brainstorming**
2. **Bug or failure?** → Start at Step 1 → invoke **debugging**
3. **Has an approved plan?** → Start at Step 4 → invoke **executing-plans** or **subagent-driven**
4. **Claiming completion?** → Step 6 → invoke **verification**
5. **Anything else that a skill covers?** → invoke it

If none apply, respond normally.

## Skill Invocation Protocol

When invoking a skill:

1. **Announce it** — "Using [skill] for [purpose]"
2. **Follow it** — the skill's process is the process. Do not improvise a different workflow
3. **Chain correctly** — each step's output feeds the next step's input. The happy path IS the chain

## Working Agreements Are Non-Negotiable

These are enforced by the scrum-master skill, but every skill respects them:

- No production code without a failing test first (TDD)
- No fix without root cause investigation (Debugging)
- No completion claim without fresh evidence (Verification)
- No design without exploring alternatives (Brainstorming)
- No merge without code review (when enabled)

## What NOT to Do

- Do not skip skill checks because the request seems simple
- Do not invoke skills you have already invoked in this conversation for the same topic — check once, not repeatedly
- Do not treat skills as optional suggestions — rigid skills are mandatory workflows
- Do not spawn sub-agents for skill invocation — invoke the skill yourself, delegate only implementation work
- Do not mix skill systems — if this project has agilar-coder skills, use them. Do not invoke plugin skills that cover the same activity
