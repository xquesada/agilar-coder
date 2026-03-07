# PBI #255: Design user-acceptance-testing skill (UAT orchestrator)

**Epic:** agilar-ai-sdlc
**Status:** Backlog

## Description

User acceptance testing is not a single activity — it's a process that varies per PBI and orchestrates multiple sub-skills depending on what the PBI requires. Currently agilar-coder has no UAT concept. The verification skill proves the *code* works; UAT proves the *feature* works from the user's perspective. These are different things.

## UAT Activities (not all apply to every PBI)

| Activity | When it applies | Sub-skill |
|----------|----------------|-----------|
| **BDD acceptance tests** | PBI has Gherkin scenarios | `bdd` |
| **Visual QA** | PBI changes UI layout, styling, components | `visual-qa` |
| **Happy-path walkthrough** | Any user-facing feature | Manual browser flow |
| **Regression testing** | Changes touch shared components or APIs | Existing test suite + spot checks |
| **Edge case exploration** | New input handling, error states, empty states | Manual or scripted |
| **Cross-viewport testing** | Responsive UI changes | `visual-qa` at multiple breakpoints |
| **Accessibility checks** | Any UI change | Lighthouse/axe or manual keyboard nav |
| **Performance smoke test** | New pages, heavy data loads | Browser timing, network tab |

## Design Questions

1. Is UAT always browser-based, or can it apply to CLI tools, APIs, background services?
2. Should UAT be a standalone skill or an extension of the verification skill?
3. How does UAT relate to the staging gate in DEVOPS.md?
4. Should the PBI format include a `## UAT` section specifying which activities apply?
5. Can the UAT skill auto-detect which activities are relevant from the PBI's acceptance criteria?

## Scope (also in this PBI)

Migrate the two global skills into agilar-coder's canonical set:
- `~/.claude/skills/visual-qa` → `skills/visual-qa/SKILL.md` (update to Playwright MCP)
- `~/.claude/skills/bdd` → `skills/bdd/SKILL.md` (replace the existing stub implementation)
- Remove global versions after migration to prevent dual-loading

## Acceptance Criteria

- [ ] Written analysis of UAT activities, when each applies, and how they compose
- [ ] Canonical `skills/user-acceptance-testing/SKILL.md` with orchestration process
- [ ] Claude Code implementation
- [ ] `visual-qa` migrated into agilar-coder canonical skills, updated to Playwright MCP
- [ ] `bdd` canonical skill updated with full content (not the current stub)
- [ ] Global `~/.claude/skills/visual-qa` and `~/.claude/skills/bdd` removed
- [ ] Scaffold updated to include visual-qa and bdd as optional skills
- [ ] UAT skill correctly selects sub-activities based on PBI acceptance criteria

## Checklist

- [ ] Analyze UAT activities and categorize by PBI type (UI, API, background service, config)
- [ ] Design UAT orchestration: how it reads AC and selects sub-skills
- [ ] Define relationship to verification skill and Definition of Done
- [ ] Define relationship to staging gate in DEVOPS.md
- [ ] Write canonical SKILL.md for `user-acceptance-testing`
- [ ] Write Claude Code implementation
- [ ] Migrate visual-qa into agilar-coder (Playwright MCP)
- [ ] Migrate bdd into agilar-coder (replace stub)
- [ ] Remove global `~/.claude/skills/visual-qa` and `~/.claude/skills/bdd`
- [ ] Update scaffold to include visual-qa and bdd as optional skills

## Notes

The UAT skill would become the final quality gate before marking a PBI done. Flow: implementation → verification (code works) → UAT (feature works for user) → code review → done. This maps to Gate 3 (Verification) in DEVOPS.md but extends it to cover the user perspective, not just test output.
