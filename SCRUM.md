# SCRUM.md — Agilar's Agile Framework for AI-Enabled Development

Agilar's adaptation of Scrum and Kanban for teams where AI agents are first-class participants. Solo developers and multi-agent setups use Kanban. Multi-human teams use full Scrum. The framework is the same — the ceremony overhead scales with coordination complexity.

## Team Modes

| Aspect | Solo | Multi-Agent | Multi-Human |
|--------|------|-------------|-------------|
| **Humans** | 1 (PO + Dev) | 1 (PO + Dev) | PO + Developer(s) |
| **Agents** | 1 agent | Orchestrator + workers | 1 agent per developer |
| **Process** | Kanban | Kanban | Full Scrum |
| **Branching** | Trunk (main) | Worktrees + main | Feature branches + main |
| **Cadence** | Continuous pull | Continuous pull | Sprint (1-2 weeks) |
| **Ceremonies** | Refinement, Retro | Refinement, Retro | All Scrum events |

---

## Roles

### Product Owner (human)

Decides WHAT to build and in what order. Owns the Product Backlog. Writes or approves acceptance criteria. In solo and multi-agent modes, this is the same person who develops.

Responsibilities:
- Maintain and prioritize the Product Backlog
- Write acceptance criteria for PBIs
- Accept or reject completed work against those criteria
- Make scope decisions — what's in, what's out, what's next

The PO is always human. An agent can coach the PO (see `skills/po-coach/`) but never replaces them.

### Scrum Master (human or agent-assisted)

Process guardian. Ensures the team follows the methodology and its working agreements. Removes impediments.

| Mode | Who fills the role |
|------|--------------------|
| **Solo** | Agent skill (`skills/scrum-master/`) acts as process conscience — nudges the human when working agreements are at risk |
| **Multi-agent** | Orchestrator agent handles process enforcement as part of task coordination |
| **Multi-human** | Dedicated human, optionally assisted by `skills/scrum-master/` agent skill |

The Scrum Master does not manage people. It manages process. In solo mode, this means the agent reminds you to write the test before the code, to verify with evidence before claiming done, and to create the PBI before starting the feature.

### Developer (human + AI pair)

Development is a pair activity. The human and agent work as a unit with a clear contract:

| Activity | Human | AI Agent |
|----------|-------|----------|
| **Brainstorm** | Drives the conversation | Proposes approaches, asks questions |
| **Decide** | Makes all decisions | Presents options with trade-offs |
| **Design** | Approves architecture | Explores codebase, drafts designs |
| **Implement** | Reviews code | Writes code following TDD |
| **Verify** | Confirms done | Runs tests, provides evidence |

The human designs. The AI implements. The human reviews. The AI verifies. Decisions always belong to the human. Process discipline always belongs to the agent.

In multi-human teams, each developer has their own agent. Agents do not share context across developers unless explicitly coordinated through the backlog or version control.

### Agent Roles in Multi-Agent Mode

In multi-agent mode, the agent team has specialized roles beyond the general developer pair:

| Role | Purpose | When Active |
|------|---------|-------------|
| **Worker** | Implements a single task following TDD. Receives a focused prompt, executes, reports back. | During subagent-driven execution |
| **Code Reviewer** | Verifies implementation against spec and code quality standards. Two modes: pre-merge review (before changes land on main) and audit (periodic codebase scan). | After each worker task, before merge |
| **CI Checker** | Monitors CI pipeline, diagnoses failures, applies fixes, verifies the fix resolves the failure. | After push to remote, when CI fails |

These roles can be filled by the same underlying agent with different prompts, or by dedicated agent definitions (see `.claude/agents/` convention in TOOLCHAIN.md). The orchestrator dispatches the right role at the right time.

Workers implement. Reviewers verify. CI checkers repair. The orchestrator coordinates all three.

---

## Artifacts

### Product Backlog

The single source of truth for all work. An ordered list of PBIs. One backlog per product — not per team, not per sprint, not per epic.

The PO owns the order. Items at the top are refined, small, and ready. Items at the bottom are rough ideas. The backlog is a living document, not a contract.

#### Where the backlog lives

The backlog is managed in the **product ownership layer**, which is separate from the development workspace. The PO refines requirements in the backlog tool; the developer (human + agent) queries the backlog from the project repo to find work.

| Backlog Tool | How the agent accesses PBIs |
|---|---|
| **Agilar PO Companion** | API — agent queries for ready PBIs at session start (recommended) |
| **Jira** | Jira REST API or CLI — agent reads ticket details by ID |
| **YAML file in repo** | Direct file read — simplest, works offline, no external dependency |
| **GitHub Issues** | `gh` CLI — agent reads issue body and labels |
| **Other** | Agent needs a documented way to fetch PBI details — URL, API, or file path |
| **Filesystem backlog** | PBI files in `backlog/` directory — coexists with any of the above as a parallel record (see Filesystem Backlog section below) |

The scaffold wizard asks which tool you use and configures the project's CLAUDE.md with the access pattern. The agent must be able to:
1. **List** available PBIs (status `ready`, filtered by project/epic)
2. **Read** a PBI's full details (notes, acceptance criteria, checklist)
3. **Update** status (mark `in_progress` when starting, `done` when complete)

If the agent cannot access the backlog, it works blind — no acceptance criteria, no Definition of Done, no way to know if the work matches what was asked for.

#### PO layer vs development layer

This separation mirrors how real teams work. The PO doesn't sit inside the IDE. The developer doesn't manage the backlog from the build system.

| Layer | What happens here | Who works here |
|---|---|---|
| **Product ownership** | Define, refine, prioritize PBIs. Write acceptance criteria. Approve designs. Track progress. | PO (human) |
| **Development** | Build, test, verify. Skills active. Working agreements enforced. CI runs. | Developer (human + agent) |

In AI-assisted development, this boundary has a technical expression: the CLAUDE.md in the development repo must include a link to the backlog system. Without it, the agent has skills and working agreements but no requirements to build against.

### Product Backlog Item (PBI)

Every PBI has:

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique identifier (auto-assigned or sequential) |
| `title` | Yes | What this PBI delivers, in plain language |
| `status` | Yes | Current lifecycle state (see PBI Lifecycle below) |
| `epic` | No | Grouping for related PBIs |
| `notes` | No | Context, background, design decisions — not action steps |
| `checklist` | No | Discrete action steps (`[{ text, done }]`). Use when a PBI has 2+ concrete steps |
| `acceptance_criteria` | Yes (for ready) | How we know this PBI is done. Testable, specific, unambiguous |

Checklist vs. notes: Action steps go in the checklist. Context and background go in notes. If you can check it off, it's a checklist item.

### Definition of Ready (DoR)

A PBI is ready to pull into work when ALL of these are true:

- **Small enough** — can be completed in a single working session (solo/multi-agent) or sprint (multi-human)
- **Acceptance criteria written** — specific, testable, agreed upon by PO
- **Design approved** — the human has reviewed and approved the approach (architecture, data model, API shape)
- **No blockers** — no dependencies on incomplete work, missing access, or unanswered questions
- **Checklist populated** — if the PBI has multiple steps, they are listed

A PBI that does not meet the DoR stays in `backlog` status. Do not pull it. Refine it first.

### Definition of Done (DoD)

A PBI is done when ALL of these are true:

- **Acceptance criteria met** — every criterion verified, not assumed
- **Tests pass** — all tests green, including new tests written via TDD (working agreement: `skills/tdd/`)
- **Code reviewed** — human has reviewed agent-written code (working agreement: `skills/code-review/`). *Optional for non-technical solo users — see scaffold onboarding.*
- **Verified with evidence** — fresh test runs, screenshots, or logs proving it works. Not "I think it works" — evidence (working agreement: `skills/verification/`)
- **Backlog tool updated** — PBI status set to `done` in the backlog tool (API, Jira, GitHub Issues, etc.). File archival is not a substitute for tool update.
- **CHANGELOG updated** — entry added under current version or `[Unreleased]` section
- **Deployed to staging** — if a staging environment exists, the change runs there before production
- **Docs updated** — if the change affects APIs, configuration, or user-facing behavior, documentation reflects it

The DoD encodes the team's working agreements. Violating any one means the PBI is not done:

| Working Agreement | DoD Gate | Skill |
|-------------------|----------|-------|
| No production code without a failing test first | Tests pass (TDD) | `skills/tdd/` |
| No completion claim without fresh verification evidence | Verified with evidence | `skills/verification/` |
| No done without backlog tool update | Backlog tool updated | `skills/verification/`, `skills/scrum-master/` |
| No merge without code review | Code reviewed (optional for non-technical solo) | `skills/code-review/` |

### Sprint Backlog (multi-human only)

The subset of the Product Backlog the team commits to for the current sprint. Exists only in multi-human mode — solo and multi-agent teams pull from the Product Backlog directly.

The Sprint Backlog is a plan, not a promise. If scope needs adjustment mid-sprint, the team negotiates with the PO. No one adds work to the Sprint Backlog without the team's agreement.

---

## Events

### All Modes

**Product Backlog Refinement**

Break down large PBIs, write acceptance criteria, estimate effort, reorder priorities. The goal is to keep the top of the backlog ready (meets DoR) at all times.

| Mode | How it works |
|------|--------------|
| **Solo** | Inline — refine and build can be a single activity. Review the backlog, pick a PBI, refine it if needed, then build |
| **Multi-agent** | Inline or scheduled — orchestrator may refine as part of task assignment |
| **Multi-human** | Scheduled session — PO presents upcoming PBIs, team discusses, refines, estimates |

**Retrospective**

What worked, what didn't, what to change. Even solo developers benefit from periodic reflection on their process.

| Mode | How it works |
|------|--------------|
| **Solo** | Periodic self-review, optionally prompted by `skills/scrum-master/`. Weekly or after major milestones |
| **Multi-agent** | Review agent performance, process adherence, working agreement violations |
| **Multi-human** | Standard Scrum retrospective at sprint end |

### Solo and Multi-Agent (Kanban)

No sprint planning. No sprint review. No daily standup. The overhead is not justified when one human coordinates with agents.

The workflow is:

1. Pull the next ready PBI from the top of the backlog
2. Refine it inline if it doesn't fully meet DoR (write acceptance criteria, break down steps)
3. Build it (following skills: TDD, verification, code review)
4. Verify it meets DoD
5. Mark it done
6. Pull the next one

Work-in-progress limit: one PBI at a time per agent. Finish before starting. Context-switching is the enemy of quality.

#### Orchestrator Responsibilities (Multi-Agent)

In multi-agent mode, the orchestrator (the agent running in the main session) has additional responsibilities:

1. **Branch-first safety** — Create a feature branch FIRST, before any worker begins implementation. Workers operate on branches, never directly on main.

2. **Worker dispatch** — Create focused, self-contained prompts for each worker. Include all context the worker needs — it has no conversation history.

3. **Review coordination** — After each worker task, dispatch the code reviewer. Spec compliance first, then code quality (see `skills/subagent-driven/`).

4. **Merge management** — After review passes, merge the worker's branch to main. Run the full test suite on main after merge.

5. **Push decision** — Push to remote only when:
   - All tests pass on main after merge
   - The human approves (or pre-approves for the session)
   - Be aware of push cost: each push may trigger CI, notify collaborators, and create a permanent record

6. **CI monitoring** — After push, check CI status. If CI fails, dispatch the CI checker agent to diagnose and fix.

#### Push Permission Model

| Scenario | Who Pushes | When |
|----------|-----------|------|
| Solo, trunk-based | Agent (after human pre-approval) | After each verified change |
| Multi-agent, worktrees | Orchestrator | After merge + test pass on main |
| Multi-human, PRs | CI/merge automation | After PR approval + CI pass |

The orchestrator never pushes broken code. The test suite must pass on main before any push.

### Multi-Human (Full Scrum)

When multiple humans coordinate, you need the full ceremony set to stay aligned.

**Sprint Planning**

- PO presents the prioritized backlog
- Team selects PBIs they can commit to for the sprint
- Each PBI must meet DoR before entering the Sprint Backlog
- Team creates a Sprint Goal — one sentence describing the sprint's purpose

**Daily Standup**

- Each developer (human + agent pair): what did we do, what will we do, any blockers
- 15 minutes max. Not a status report — a coordination event
- Agents can prepare standup summaries for their human

**Sprint Review**

- Team demonstrates completed PBIs to stakeholders
- PO accepts or rejects against acceptance criteria
- Feedback captured as new or updated PBIs

**Sprint Retrospective**

- What went well, what didn't, what to improve
- Include agent-specific topics: skill effectiveness, working agreement adherence, agent coordination issues
- One actionable improvement per retrospective, minimum

---

## PBI Lifecycle

```
backlog ──→ ready ──→ in_progress ──→ done
  │           │            │
  │           │            └── fails DoD → stays in_progress, fix issues
  │           │
  │           └── blocker found → back to backlog
  │
  └── refined + meets DoR → ready
```

### Status Definitions

| Status | Meaning | Gate |
|--------|---------|------|
| `backlog` | Identified work, not yet refined | None |
| `ready` | Refined, meets Definition of Ready | DoR |
| `in_progress` | Actively being worked on | Pulled by developer |
| `done` | Complete, meets Definition of Done | DoD |

### Transitions

**backlog to ready**: PBI has been refined. Acceptance criteria are written. Design is approved. It's small enough. No blockers. The PO confirms it meets the DoR.

**ready to in_progress**: A developer (human + agent pair) pulls the PBI. In Kanban modes, this is the next item at the top. In Scrum mode, it was committed during Sprint Planning.

**in_progress to done**: All DoD criteria are met. Tests pass. Code is reviewed. Verification evidence exists. The PO accepts it against the acceptance criteria.

**in_progress stays in_progress**: If the PBI fails any DoD gate, it stays in progress. Fix the issue. Do not move it to done and create a "fix" PBI — that's cheating.

**ready back to backlog**: A blocker was discovered, or priorities changed. The PBI needs more refinement or is no longer relevant for now.

---

## The PBI-First Rule

**Any new work gets a PBI before starting.**

This is not optional. This is not "when convenient." Before you write a line of code, there is a PBI for it.

Specifically:
- New UI page? PBI first.
- New API endpoint? PBI first.
- New cron job, tool, or script? PBI first.
- Bug fix? PBI first.
- Refactoring? PBI first.
- Research spike? PBI first.

If the human forgets to create a PBI, the agent suggests it. This is part of the Scrum Master conscience (see `skills/scrum-master/`).

Why: Without a PBI, there is no acceptance criteria. Without acceptance criteria, there is no Definition of Done. Without DoD, there is no way to know if you're finished. You will gold-plate, scope-creep, or ship something that doesn't match what was needed.

The PBI is the contract between the PO and the developer. No contract, no work.

---

## Filesystem Backlog

Every project has a `backlog/` directory. When an external backlog tool is configured, the tool is the **single source of truth for PBI status** — folders are archival organization, not status tracking. When no external tool exists (standalone mode), folder position IS the status.

**Key rule:** Update the tool first, then move the file. Never move a file to `done/` without updating the tool. If the tool is unreachable, add `## API Sync Needed` to the PBI file and reconcile next session.

### Folder Structure

```
backlog/
├── pbi-042-email-validation.md         # status: backlog (unrefined)
├── pbi-043-user-profile.md             # status: backlog
├── ready/
│   ├── pbi-044-search-feature.md       # refined, planned, approved for execution
│   └── pbi-045-notification-api.md
├── in_progress/
│   └── pbi-046-login-page.md           # being worked on
└── done/
    ├── pbi-040-setup.md                # completed
    └── pbi-041-homepage.md
```

### PBI File Naming

`pbi-NNN-short-description.md` where NNN matches the PBI ID from the backlog tool (or is auto-assigned sequentially when no external tool exists).

### PBI File Format

A PBI file evolves through its lifecycle. Not all sections exist at creation — they accumulate as the PBI is refined and planned.

**Required header:**

```markdown
# PBI #NNN: Title
```

**Optional sections** (added as the PBI matures through the lifecycle):

```markdown
# PBI #42: Email Validation

**Epic:** user-management

## Description
Add email validation to the user registration form.

## Design
(Added during brainstorming — Phase 4 output lives here, not in a separate file)

**Goal:** Validate email format on registration to reduce bad data.

**Approach:** Simple regex pattern matching, not RFC 5322 full compliance.

**Design concerns scan:** Architecture (not relevant), Data model (not relevant),
API (relevant — new validation on POST /users), Security (relevant — input validation),
Performance (not relevant), UI (not relevant — backend only).

**API:** POST /users returns 400 with `{"error": "invalid_email"}` when format fails.

**NFR checkpoint:** Security — validates input before processing. No other NFRs relevant.

**Out of scope:** Email deliverability checks, MX record validation.

## Acceptance Criteria
- [ ] validate_email returns False for invalid emails
- [ ] validate_email returns True for valid emails
- [ ] API returns 400 for invalid email on registration

## Plan
(Added during sprint-planning — task breakdown)

### Task 1: Create validation tests (red)
**File(s):** tests/test_user_validation.py
**What to do:** ...
**Test:** ...
**Pre-commit:** ...
**Commit:** "test: add email validation tests (red)"

### Task 2: Implement validation (green)
**File(s):** src/user_validation.py
**What to do:** ...

## Notes
(Context, decisions, discoveries during implementation)
We discussed using regex — simple pattern matching, not RFC 5322 full compliance.
```

The PBI file is a **living document** that grows through the lifecycle. Each step in the happy path appends to it:

| Step | Section added |
|------|--------------|
| Capture | `# PBI #NNN: Title` + `## Description` |
| Refine (brainstorming) | `## Design` + `## Acceptance Criteria` |
| Plan (sprint-planning) | `## Plan` |
| Implement | `## Notes` (decisions, discoveries) |
| Close | Final state archived in `backlog/done/` |

**Design lives in the PBI file, not in a separate document.** Cross-cutting architecture decisions that span multiple PBIs belong in `docs/architecture/` as ADRs.

### PBI File Lifecycle

```
Work identified
      │
      v
PBI created in backlog tool → get PBI number
      │
      v
PBI file created → backlog/pbi-NNN-description.md (title + description)
      │
      v
Tool updated → reference to file path added in notes
      │
      v
Refined → ## Design + ## Acceptance Criteria added to file
      │
      v
Planned → ## Plan section added (tasks from sprint-planning skill)
      │
      v
Approved → file moved to backlog/ready/, tool status updated
      │
      v
Picked up → file moved to backlog/in_progress/, tool status updated
      │
      v
Completed → file moved to backlog/done/, tool status updated
```

### Sync Protocol (with external tools)

When an external backlog tool is configured, it is the **primary source of truth**. The filesystem is the **detailed record**. Two sources, clear roles:

| Source | Role | Contains |
|--------|------|----------|
| **Backlog tool** | Primary — status, priority, metadata | Title, status, epic, short notes, checklist |
| **PBI file** (`backlog/`) | Detail — the full living document | Description, design, plan, AC, implementation notes |

#### The Sync Workflow

Every PBI interaction follows this order:

1. **Check the tool first.** Before creating a file, query the backlog tool. Does a PBI already exist for this work?
2. **Create in the tool first.** If no PBI exists, create it in the backlog tool. Get the PBI number.
3. **Create the file.** Create `backlog/pbi-NNN-description.md` using the PBI number from the tool.
4. **Reference the file from the tool.** Update the PBI in the tool with a reference to the file path (in the notes field).
5. **The file links back automatically.** The filename contains the PBI number (`pbi-NNN-*`), so the link is always there.
6. **Close in the tool first.** When a PBI is done, update the tool status to `done` BEFORE moving the file to `backlog/done/`. Tool update is part of the Definition of Done. If the tool is unreachable, do not move the file — add `## API Sync Needed` to the PBI file and reconcile when connectivity is restored.

#### Conflict Resolution

- **If tool and file disagree, the tool wins.** The agent updates the file to match.
- **If a PBI number changes in the tool**, the file must be renamed to match.
- **If a PBI is deleted in the tool**, the file should be moved to `backlog/done/` or removed.

#### Compatible Backlog Tools

A "compatible" backlog tool is one that is **agilar-coder aware** — it understands the filesystem backlog exists and proactively keeps it in sync.

A compatible tool:
- Notifies the agent (via webhook, file watch, or API) when a PBI is created, updated, renamed, or deleted
- Propagates PBI number changes to the corresponding file (rename `pbi-NNN-*` → `pbi-MMM-*`)
- Propagates status changes to the file's folder location (`backlog/` → `ready/` → `in_progress/` → `done/`)
- Propagates deletion (moves file to `done/` or removes it)

A non-compatible tool works fine — the agent handles sync manually using the workflow above. Compatible tools simply automate what the agent would do anyway.

| Tool | Compatible | Notes |
|------|-----------|-------|
| **Agilar PO Companion** | Yes (planned) | Native integration with filesystem backlog |
| **Jira** | No | Agent syncs manually via REST API |
| **GitHub Issues** | No | Agent syncs manually via `gh` CLI |
| **YAML file in repo** | N/A | File IS the tool — no sync needed |

### Without External Tool

When no external tool is configured, `backlog/` IS the backlog. Git history is the audit trail. No sync needed — the filesystem is the single source of truth. PBI numbers are auto-assigned sequentially.

---

## Working Agreements

The working agreements defined in the skills are not separate from the Agile framework — they are built into it. The DoD is the enforcement mechanism.

### TDD

> No production code without a failing test first.

Enforced via DoD: "Tests pass" means tests were written first (red-green-refactor), not bolted on after. The agent follows `skills/tdd/` rigorously. Code review (when enabled) confirms TDD discipline was followed.

### Verification

> No completion claim without fresh verification evidence.

Enforced via DoD: "Verified with evidence" means the agent ran the tests, captured the output, and presented it. Not "the tests should pass" — actual evidence. Screenshots, test output, log snippets. The agent follows `skills/verification/` to produce this evidence before marking a PBI done.

### Code Review (optional for non-technical solo users)

> No merge without code review.

Enforced via DoD when enabled: "Code reviewed" means the human reviewed agent-written code before it ships. In solo mode, this is the developer reviewing their agent's output. In multi-human mode, this includes peer review. The process follows `skills/code-review/`.

**When to skip:** If you're a non-technical PO working in solo mode, you may not be able to review code meaningfully. The scaffold wizard asks about this during onboarding. When code review is disabled, the agent relies on TDD + verification + automated quality checks (linting, type checking) as the quality safety net.

### Debugging

> No fix without root cause investigation.

Not a DoD gate directly, but enforced by `skills/debugging/`. When a bug is found, the agent does not guess-and-patch. It investigates root cause through a structured four-phase process before proposing a fix. This prevents the "fix one bug, create two" cycle.

### Brainstorming

> No design without exploring alternatives.

Enforced during refinement. Before committing to a design, the human and agent explore alternatives via `skills/brainstorming/`. The PBI's `## Design` section reflects that alternatives were considered.

---

## Kanban vs. Scrum: Decision Guide

Use this to pick the right mode for your team:

| Question | If yes → | If no → |
|----------|----------|---------|
| More than one human developer? | Multi-human (Scrum) | Solo or Multi-agent |
| Multiple agents working in parallel on different PBIs? | Multi-agent (Kanban) | Solo (Kanban) |
| Need sprint commitments for stakeholder alignment? | Multi-human (Scrum) | Kanban |
| One person doing everything? | Solo (Kanban) | Multi-human or Multi-agent |

You can evolve between modes. A solo developer who adds a second agent moves to multi-agent. A multi-agent setup that brings on a second human moves to multi-human. The backlog, PBI structure, and working agreements stay the same — only the ceremonies change.
