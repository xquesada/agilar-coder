# Using Agilar Coder: Session Bootstrapper

Every session starts here. This skill ensures the right skills are invoked at the right time. Without it, skills load passively and may be ignored.

## The Rule

**Check for applicable skills BEFORE any response or action.** If a skill might apply, invoke it. If it turns out to be wrong for the situation, you don't need to follow it.

## Skill Inventory

These are the skills available in an agilar-coder project. Know them by name and purpose:

### Process Skills (determine HOW to approach)

| Skill | When to invoke |
|-------|---------------|
| brainstorming | Before any creative work: new features, architecture changes, designs, decisions |
| sprint-planning | When you have requirements and need to break work into implementation steps |
| executing-plans | When you have a written plan to execute across sessions |
| scrum-master | Always on. Enforces working agreements, PBI-first, DoR/DoD gates |
| facilitator | When running Scrum events: refinement, planning, standup, review, retro |

### Engineering Skills (guide execution)

| Skill | When to invoke |
|-------|---------------|
| tdd | Before implementing any feature or bugfix |
| debugging | When encountering any bug, test failure, or unexpected behavior |
| bdd | When adding features with acceptance criteria that benefit from Given/When/Then |
| verification | Before claiming work is complete, before committing |

### Code Review Skills

| Skill | When to invoke |
|-------|---------------|
| code-review | After completing tasks, before merging |
| requesting-code-review | When requesting review from a human or another agent |
| receiving-code-review | When receiving review feedback, before implementing suggestions |

### Multi-Agent Skills

| Skill | When to invoke |
|-------|---------------|
| subagent-driven | When executing plans with independent tasks in the current session |
| parallel-agents | When facing 2+ independent tasks without shared state |
| git-worktrees | When starting feature work that needs isolation |

### Completion Skills

| Skill | When to invoke |
|-------|---------------|
| finishing-a-development-branch | When work on a branch or worktree is complete |

### Coaching Skills

| Skill | When to invoke |
|-------|---------------|
| po-coach | During backlog refinement, acceptance criteria writing, prioritization |

## The Happy Path: Idea to Production

This is the default step-by-step for taking a raw idea to a fully done feature. Not every PBI needs every step — scale the ceremony to the complexity. But this is the default, and skipping steps is a conscious choice, not an accident.

### Step 1: Capture — PBI exists
Every piece of work starts as a PBI. Follow the sync workflow:
1. Check the backlog tool — does a PBI already exist?
2. If not, create the PBI in the backlog tool first. Get the PBI number.
3. Create the PBI file: `backlog/pbi-NNN-description.md`
4. Reference the file from the tool (add file path to notes).
- **Skill:** scrum-master (PBI-first rule, always on)

### Step 2: Refine — understand what "done" means
Turn the raw PBI into something buildable. Write acceptance criteria, scan design concerns, explore approaches, get design approval. The PBI must meet Definition of Ready before implementation starts.
- **Skill:** po-coach (acceptance criteria, scope, prioritization)
- **Skill:** brainstorming (design concerns scan, propose 2-3 alternatives, get design approval)
- **Output:** `## Design` and `## Acceptance Criteria` sections added to the PBI file (not a separate design doc)

### Step 3: Plan — break it into tasks
Take the approved design and break it into ordered, testable implementation steps. Each task should be small enough to complete in one TDD cycle.
- **Skill:** sprint-planning (task breakdown, execution mode recommendation)
- **Output:** `## Plan` section added to the PBI file

### Step 4: Implement — build it with TDD
Execute the plan task by task. Each task follows the red-green-refactor cycle. If a bug is found during implementation, switch to debugging before fixing.
- **Skill:** tdd (per task — write failing test, make it pass, refactor)
- **Skill:** debugging (if something breaks — root cause first, then fix)
- **Skill:** bdd (if acceptance criteria benefit from Given/When/Then)
- **Execution mode:** executing-plans (solo), subagent-driven (multi-agent), or parallel-agents (independent tasks)

### Step 5: Review — verify the work
Before merging, review the code for spec compliance and quality. When code review is enabled, request it. When receiving feedback, evaluate it critically.
- **Skill:** requesting-code-review (dispatch reviewers)
- **Skill:** receiving-code-review (respond to feedback)

### Step 6: Verify — prove it works
Run the full test suite, linter, and build. Read the output. Confirm all acceptance criteria are met with fresh evidence. No stale results.
- **Skill:** verification (evidence before claims)

### Step 7: Finish — merge and clean up
Decide what to do with the branch: merge, push for PR, keep, or discard. Clean up worktrees. Post-merge: run tests again on the merged result.
- **Skill:** finishing-a-development-branch (merge/PR/keep/discard)

### Step 8: Deploy — ship it to production
If the project has a deployment target, deploy the merged code. Verify the service starts, health checks pass, and the feature works in production. Update any deployment docs or runbooks if the deploy process changed.
- **How:** Follow the project's deploy workflow in CLAUDE.md
- **Verify:** Health check, smoke test, or manual confirmation from the human partner

### Step 9: Close — mark done, update changelog, capture knowledge
Move the PBI to done. Update CHANGELOG.md with what changed. Suggest a version bump based on the version impact classification from the design phase. The human confirms or overrides before the bump happens.
- **Skill:** scrum-master (DoD gate — all criteria met before marking done)
- **CHANGELOG:** Add entry under current version or `[Unreleased]` (Added/Changed/Fixed/Removed)
- **Version bump:** Suggest major/minor/patch based on design classification. Human confirms.
- **Update:** CLAUDE.md, project docs, memory files as needed

### Shortcut Paths

Not every PBI goes through all 9 steps. Common shortcuts:

| Scenario | Path |
|----------|------|
| Bug fix | Step 1 → Step 4 (debugging → tdd) → Step 6 → Step 7 → Step 8 |
| Config change / trivial fix | Step 1 → Step 4 (tdd if testable) → Step 6 → Step 7 |
| Research / investigation | Step 1 → Step 2 (brainstorming) → Step 9 (capture findings) |
| Existing plan, ready to build | Step 4 → Step 5 → Step 6 → Step 7 → Step 8 → Step 9 |

The scrum-master skill is always on and will nudge if you skip a step that matters.

## Skill Priority

When multiple skills could apply, invoke them in this order:

1. **Process skills first** — brainstorming, debugging, sprint-planning
2. **Engineering skills second** — tdd, bdd, verification
3. **Coaching skills as needed** — po-coach, facilitator

## Red Flags

These thoughts mean STOP — you are rationalizing skipping a skill:

| Thought | Reality |
|---------|---------|
| "This is too simple for a skill" | Simple tasks are where unexamined assumptions waste time |
| "Let me just start coding" | TDD and brainstorming exist precisely for this impulse |
| "I already know the approach" | Present alternatives anyway. The human partner may disagree |
| "I'll check skills after this one thing" | Check BEFORE doing anything |
| "The skill is overkill" | A two-minute design check is never overkill |
| "I need more context first" | Skills tell you HOW to gather context |

## Skill Types

**Rigid skills** — follow exactly, do not adapt away the discipline:
- tdd (red-green-refactor cycle)
- debugging (root cause investigation before fix)
- verification (evidence before completion claims)
- brainstorming (design approval before code)

**Flexible skills** — adapt principles to context:
- sprint-planning, facilitator, po-coach, bdd

**Always-on skills** — run as background checks, not invoked explicitly:
- scrum-master

The skill itself tells you which type it is.

## User Instructions

Instructions say WHAT, not HOW. "Add X" or "Fix Y" does not mean skip workflows. The working agreements apply regardless of how the request is phrased.
