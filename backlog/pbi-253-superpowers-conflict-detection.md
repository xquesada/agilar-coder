# PBI #253: Superpowers conflict detection and removal in CLI

**Epic:** agilar-ai-sdlc
**Status:** Backlog

## Description

The Anthropic superpowers plugin (v4.3.1) loads globally via `~/.claude/plugins/` and silently overrides agilar-coder repo skills in every Claude Code session. 10 of 17 agilar-coder skills have a superpowers equivalent with conflicting instructions (different plan placement, no PBI awareness, different execution handoff, worktree-by-default).

Users must choose one: Anthropic Superpowers or Agilar Coder. The CLI needs to enforce this at installation time, assess time, and runtime.

## Context

Detection paths:
- Plugin cache: `~/.claude/plugins/cache/claude-plugins-official/superpowers/`
- Plugin config: `claude` CLI plugin commands (if available)
- Session context: superpowers skill names use `superpowers:` prefix

Conflicting skill pairs (superpowers -> agilar-coder):
- `superpowers:brainstorming` -> `brainstorming`
- `superpowers:writing-plans` -> `sprint-planning`
- `superpowers:executing-plans` -> `executing-plans`
- `superpowers:test-driven-development` -> `tdd`
- `superpowers:systematic-debugging` -> `debugging`
- `superpowers:verification-before-completion` -> `verification`
- `superpowers:requesting-code-review` -> `requesting-code-review`
- `superpowers:receiving-code-review` -> `receiving-code-review`
- `superpowers:using-git-worktrees` -> `git-worktrees`
- `superpowers:finishing-a-development-branch` -> `finishing-a-development-branch`

## Acceptance Criteria

- [ ] `agilar-coder install` detects superpowers and warns with clear explanation
- [ ] `agilar-coder assess` includes a "No conflicting plugins" dimension (dimension 9)
- [ ] Assess scores 0 on dimension 9 if superpowers is present, 100 if clean
- [ ] scrum-master skill warns at runtime if superpowers skills are detected in session
- [ ] README.md documents the "choose one" decision with rationale
- [ ] User can proceed with install even if superpowers is present (warning, not blocker)

## Checklist

- [ ] Add `detect_superpowers()` function to agilar-coder CLI
- [ ] Integrate detection into `cmd_install()` — warn and offer removal guidance
- [ ] Integrate detection into `cmd_assess()` as dimension 9
- [ ] Update scrum-master skill (canonical + implementation) with runtime conflict warning
- [ ] Add "choose one" documentation to README.md
- [ ] Test: install superpowers, run assess, verify it flags the conflict
- [ ] Test: remove superpowers, run assess, verify clean score

## Notes

The CLI should NOT auto-remove the plugin — that's destructive. It should explain the conflict and tell the user how to remove it themselves. The assess dimension makes the conflict visible in every compliance check.
