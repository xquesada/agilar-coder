---
name: using-agilar-coder
description: "Session bootstrapper — invoke BEFORE any response or action. Establishes which skills to use and when. If a skill might apply, follow its instructions first."
---

<!-- Adapted from Claude Code: skill invocation via Skill tool replaced with direct instruction following. Tool-specific references replaced with generic actions. -->

# Using Agilar Coder — Codex Implementation

> Canonical skill: `skills/using-agilar-coder/SKILL.md`
>
> This file adds Codex-specific behavior. The skill inventory, priority rules, and red flags in the canonical skill apply without modification.

## How Skills Work in Codex

Skills are markdown files that describe processes. They may be provided as part of the agent's instructions, live in the project's skill directory, or be referenced in CLAUDE.md. When a skill applies, read it and follow its instructions directly.

## The Happy Path: Idea to Production

The canonical skill defines a 9-step default lifecycle. Here is how each step maps to Codex actions:

| Step | Skill | Codex Action |
|------|-------|--------------|
| 1. Capture | scrum-master | Check backlog tool → create PBI in tool → create `backlog/pbi-NNN.md` → reference file from tool |
| 2. Refine | po-coach, brainstorming | Follow skill instructions. Add `## Design` + `## Acceptance Criteria` to PBI file |
| 3. Plan | sprint-planning | Follow skill instructions. Add `## Plan` to PBI file |
| 4. Implement | tdd, debugging, bdd | Follow skill per task. Use executing-plans or subagent-driven for multi-task |
| 5. Review | requesting-code-review | Self-review: spec compliance then code quality (see subagent-driven skill) |
| 6. Verify | verification | Run test/lint/build in terminal, read full output |
| 7. Finish | finishing-a-development-branch | Merge, clean up worktrees, post-merge test |
| 8. Deploy | (project-specific) | Follow CLAUDE.md deploy workflow in terminal |
| 9. Close | scrum-master | Move PBI to `backlog/done/`, update CHANGELOG.md, suggest version bump, update docs/CLAUDE.md |

**When a user says "build this"**, the default path is: Step 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9. Scale ceremony to complexity — a bug fix skips Steps 2-3, a config change skips Steps 5-7. But the default is the full path.

## Session Start Behavior

On every new message from the user, determine where in the happy path this request falls:

1. **New idea / feature / change?** → Start at Step 1 (capture) or Step 2 (refine) → follow **brainstorming** instructions
2. **Bug or failure?** → Start at Step 1 → follow **debugging** instructions
3. **Has an approved plan?** → Start at Step 4 → follow **executing-plans** or **subagent-driven** instructions
4. **Claiming completion?** → Step 6 → follow **verification** instructions
5. **Anything else that a skill covers?** → follow it

If none apply, respond normally.

## Skill Invocation Protocol

When following a skill:

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
- Do not follow a skill you have already followed in this conversation for the same topic — check once, not repeatedly
- Do not treat skills as optional suggestions — rigid skills are mandatory workflows
- Do not mix skill systems — if this project has agilar-coder skills, use them. Do not follow other skill systems that cover the same activity
