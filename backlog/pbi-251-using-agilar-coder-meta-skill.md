# PBI #251: Create using-agilar-coder meta-skill (session bootstrapper)

**Epic:** agilar-ai-sdlc
**Status:** Backlog

## Description

The Anthropic superpowers plugin includes a `using-superpowers` meta-skill that aggressively hijacks every Claude Code session, directing the agent to invoke superpowers skills instead of repo-level agilar-coder skills. Agilar-coder has no equivalent — its repo skills load passively into context and get silently overridden.

This is the root cause of the methodology being ignored: without a session bootstrapper, the 17 agilar-coder skills are invisible background text while the superpowers plugin actively commands the agent.

## Context

The superpowers `using-superpowers` skill:
- Loads at session start with extremely aggressive language ("ABSOLUTELY MUST invoke")
- Directs Claude to invoke `superpowers:*` skills by exact name
- Chains skills: `brainstorming` -> `writing-plans` -> `executing-plans`
- Once the agent enters the superpowers pipeline, it never touches repo skills

Agilar-coder needs its own bootstrapper that:
- Is equally assertive about invoking agilar-coder skills
- References the correct skill names (`brainstorming`, `sprint-planning`, `executing-plans`, etc.)
- Enforces the agilar methodology: PBI-first, `backlog/` lifecycle, design docs vs plans distinction
- Explicitly states that repo skills override plugin skills for the same activity

## Acceptance Criteria

- [ ] Canonical `skills/using-agilar-coder/SKILL.md` exists with tool-agnostic process definition
- [ ] Claude Code implementation `implementations/claude-code/using-agilar-coder.md` exists
- [ ] Scaffold installs the skill into `.claude/skills/` for new projects
- [ ] Skill correctly chains: `brainstorming` -> `sprint-planning` -> `executing-plans`
- [ ] Skill references the PBI-first rule and `backlog/` lifecycle
- [ ] Fresh session in chepibe-core invokes agilar-coder skills, not superpowers equivalents
- [ ] CLAUDE.md updated to list the skill count as 18 (was 17)

## Checklist

- [ ] Read superpowers `using-superpowers` SKILL.md as reference
- [ ] Draft `using-agilar-coder` canonical SKILL.md in `skills/using-agilar-coder/`
- [ ] Write Claude Code implementation in `implementations/claude-code/using-agilar-coder.md`
- [ ] Add to scaffold template so new projects get it automatically
- [ ] Install into chepibe-core for validation
- [ ] Test: start fresh session in chepibe-core, verify agilar-coder skills are invoked

## Notes

Design doc: N/A (straightforward adaptation of existing pattern).
Reference: `~/.claude/plugins/cache/claude-plugins-official/superpowers/4.3.1/skills/using-superpowers/SKILL.md`
