# PBI #254: Research: Agilar Coder as a Claude Code plugin (architecture analysis)

**Epic:** agilar-ai-sdlc
**Status:** Backlog

## Description

Agilar-coder currently installs skills into `.claude/skills/` at the repo level. The Anthropic superpowers loads as a Claude Code plugin at the user level (`~/.claude/plugins/`). Plugins have higher loading precedence in the session context — this is the architectural reason superpowers overrides repo skills.

Analyze what it would mean to distribute agilar-coder (or parts of it) as a Claude Code plugin, and whether that's desirable.

## Questions to Answer

1. **Plugin format**: What does a Claude Code plugin consist of? (manifest.json, skill structure, hooks, commands, agents)
2. **Coexistence**: Can a plugin coexist with repo-level `.claude/skills/`? Which takes precedence?
3. **Distribution model**: Would plugin distribution replace or complement the current `install`/`upgrade` CLI?
4. **Customization**: How do plugins handle project-specific config (backlog adapters, external tools)?
5. **Conflict prevention**: Can a plugin enforce that conflicting plugins are not installed?
6. **Marketplace**: What's the plugin marketplace model? Can agilar-coder be listed?
7. **Hybrid model**: Could the meta-skill + bootstrapper be a plugin while project-specific skills stay repo-level?

## Tradeoffs to Evaluate

| Aspect | Plugin | Current (repo-level) |
|--------|--------|---------------------|
| Installation | One install for all repos | Per-repo install via CLI |
| Updates | Auto-update (plugin system) | Manual `upgrade` command |
| Precedence | User-level (wins over repo) | Repo-level (loses to plugins) |
| Customization | Constrained by plugin format | Full control per project |
| Vendor lock-in | Depends on Claude Code plugin system | Works with any tool that reads `.claude/skills/` |
| Distribution | Plugin marketplace | Git clone + CLI |

## Acceptance Criteria

- [ ] Written analysis covering all 7 questions above
- [ ] Tradeoff table with clear pros/cons for each approach
- [ ] Explicit recommendation with reasoning
- [ ] Analysis saved to `docs/` in agilar-coder repo

## Checklist

- [ ] Study Claude Code plugin format: manifest.json, skill loading, hooks, commands, agents
- [ ] Study superpowers plugin structure as reference implementation
- [ ] Analyze precedence: plugin skills vs repo `.claude/skills/` — which wins?
- [ ] Evaluate hybrid model: plugin for meta-skill, repo for project-specific skills
- [ ] Document tradeoffs: distribution, updates, customization, vendor dependency
- [ ] Write recommendation with pros/cons
- [ ] Save analysis to `docs/` in agilar-coder repo

## Notes

This is a research PBI — output is a written analysis with recommendation, not code. The decision will inform whether agilar-coder evolves into a plugin, stays repo-level, or goes hybrid.
