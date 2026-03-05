# PBI #252: Fix skill audit findings (sprint-planning, BDD, naming)

**Epic:** agilar-ai-sdlc
**Status:** Backlog

## Description

A full audit of all 17 canonical skills vs their Claude Code implementations (2026-03-05) found 3 issues. All implementations are byte-identical to their installed copies in chepibe-core (sync is clean), so fixes to implementations propagate cleanly.

### Finding 1: sprint-planning.md line 18 (MEDIUM)

The Claude Code implementation says:

> "Save the plan to a file. Write it to `docs/plans/` or a location the human specifies."

The canonical SKILL.md (line 57) correctly says:

> "The implementation plan is written as a `## Plan` section inside the PBI file at `backlog/pbi-NNN-short-description.md`."

This is a direct cause of plans landing in `docs/plans/` instead of PBI files.

### Finding 2: bdd.md is a stub (MEDIUM)

Only 69 lines vs 200 in canonical. Missing: Gherkin syntax examples, Scenario Outline convention, Background convention, tags for organization, common mistakes section, red flags section, verification checklist. An agent reading only the implementation gets almost no guidance on BDD practices.

### Finding 3: git-worktrees.md front-matter (LOW)

`name: using-git-worktrees` in front-matter, but canonical directory and filename both use `git-worktrees`. Creates inconsistency in skill invocation.

## Acceptance Criteria

- [ ] sprint-planning.md line 18 references `backlog/pbi-NNN.md`, not `docs/plans/`
- [ ] bdd.md implementation covers all sections from canonical SKILL.md
- [ ] git-worktrees.md front-matter `name` field matches `git-worktrees`
- [ ] `agilar-coder assess` on chepibe-core still passes after changes
- [ ] Updated skills installed in chepibe-core

## Checklist

- [ ] Fix sprint-planning.md line 18: replace `docs/plans/` with PBI file reference
- [ ] Rewrite bdd.md to match canonical coverage (Gherkin, Scenario Outline, Background, tags, common mistakes, red flags, verification checklist)
- [ ] Fix git-worktrees.md front-matter name field to `git-worktrees`
- [ ] Run `agilar-coder assess` on chepibe-core to verify compliance
- [ ] Re-install updated skills into chepibe-core

## Notes

Audit conducted in chepibe session 2026-03-05. All 14 other skills passed clean — no contradictions, no legacy references, no missing content, no sync drift.
