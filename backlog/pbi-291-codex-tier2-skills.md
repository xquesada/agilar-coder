# PBI #291: Codex skill implementations — Tier 2 (adaptation required)

**Epic:** agilar-ai-sdlc
**Status:** Backlog

## Description

Port the 4 skills that rely on Claude-specific execution patterns to Codex format. These need adaptation, not literal translation.

### Skills to port (4)

| Skill | Claude-specific dependency | Adaptation needed |
|-------|---------------------------|-------------------|
| subagent-driven | Claude Agent tool, subagent spawning | Codex doesn't have subagents — restructure as sequential workflow or document Codex's own orchestration if available |
| parallel-agents | Claude parallel Task/Agent execution | Same as above — may need to become sequential or use shell-level parallelism |
| git-worktrees | Claude worktree-aware agent spawning | Core git-worktree mechanics are tool-agnostic; strip the Claude agent orchestration layer |
| using-agilar-coder | References Claude CLI, Claude-specific flags | Adapt to reference Codex CLI, Codex-specific invocation patterns |

### Format

Same as Tier 1: flat files at `implementations/codex/<skill-name>.md` with YAML frontmatter.

### Porting approach

For each skill:
1. Identify what is tool-agnostic process (keep) vs Claude-specific mechanism (adapt)
2. Find the Codex-native equivalent or simplify the workflow
3. Document what was changed and why in a comment block at the top of the file
4. If a skill concept doesn't translate at all, document that clearly rather than forcing a bad port

## Acceptance Criteria

- [ ] `implementations/codex/` has 4 additional `.md` files (18 total with Tier 1)
- [ ] Each file has valid YAML frontmatter with `name` and `description`
- [ ] Each adapted skill documents what changed from the Claude Code version and why
- [ ] `using-agilar-coder` correctly references Codex CLI invocation
- [ ] No Claude-specific assumptions remain
- [ ] Skills that can't fully translate document their limitations honestly

## Dependencies

- Should be done after PBI #290 (Tier 1) to establish the Codex implementation pattern

## Notes

- `using-agilar-coder` is critical to the framework — it teaches the agent how to use agilar-coder itself. This must work well in Codex.
- If Codex gains orchestration capabilities in the future, these skills can be updated
