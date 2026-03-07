# PBI #290: Codex skill implementations — Tier 1 (direct ports)

**Epic:** agilar-ai-sdlc
**Status:** Backlog

## Description

Port the 14 workflow-focused canonical skills to Codex format as flat files under `implementations/codex/`. These skills are mostly process instructions and port cleanly without major adaptation.

### Format

Each skill is a single file: `implementations/codex/<skill-name>.md`

File structure:
```markdown
---
name: <skill-name>
description: <when to use this skill — must be specific enough to trigger reliably>
---

# <Skill Title> — Codex Implementation

> Canonical skill: `skills/<name>/SKILL.md`

<instructions adapted for Codex>
```

### Porting rules

1. Replace Claude-specific tool references with Codex equivalents (terminal commands, `apply_patch` for edits)
2. Remove references to Claude subagents, Task tool, Agent tool, etc.
3. Keep instructions concise — Codex works best with shorter, focused skill files
4. Write strong `description` fields that clearly state when the skill should activate
5. Reference the canonical SKILL.md as source of truth (same pattern as Claude Code implementations)

### Skills to port (14)

| Skill | Notes |
|-------|-------|
| tdd | Core workflow skill, straightforward port |
| debugging | Replace Claude tool references |
| verification | Process-focused, clean port |
| code-review | Process-focused, clean port |
| requesting-code-review | Process-focused, clean port |
| receiving-code-review | Process-focused, clean port |
| brainstorming | Process-focused, clean port |
| sprint-planning | Process-focused, clean port |
| executing-plans | Process-focused, clean port |
| bdd | Process-focused, clean port |
| finishing-a-development-branch | Git workflow, replace any Claude-specific commit/push patterns |
| scrum-master | Facilitation skill, clean port |
| po-coach | Facilitation skill, clean port |
| facilitator | Facilitation skill, clean port |

## Acceptance Criteria

- [ ] `implementations/codex/` directory exists with 14 `.md` files
- [ ] Each file has valid YAML frontmatter with `name` and `description`
- [ ] Descriptions are specific and actionable (not generic one-liners)
- [ ] No Claude-specific tool names, agent primitives, or assumptions remain
- [ ] Each file references its canonical SKILL.md
- [ ] A reviewer can read any Codex skill and understand the complete workflow without needing to also read the Claude Code version

## Notes

- Use the Claude Code implementations as a reference for structure and tone, but don't copy-paste — adapt for Codex's execution model
- When in doubt about a Codex equivalent, use generic terminal/shell instructions rather than guessing at Codex-specific APIs
