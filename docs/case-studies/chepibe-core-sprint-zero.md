# Case Study: Sprint Zero with the Agilar AI SDLC

**Project:** chepibe-core — Go engine for a personal AI operations platform
**Date:** 2026-03-01
**Duration:** Single session (~25 minutes wall clock)
**Team mode:** Solo (1 human PO + 1 AI agent, Kanban, trunk-based)
**Stack:** Go 1.26, Claude Code (Claude Opus 4.6)

---

## What is this document?

A factual account of using the Agilar AI SDLC methodology to take a Go microservices project from zero to development-ready in a single working session. No code was written beyond scaffolding. The output was a complete development environment, CI pipeline, working agreements, and a prioritized product backlog — everything a team needs to pull the first PBI and start building.

This is Sprint Zero: the work that happens before the first line of production code.

---

## Starting Point

### The product vision

Xavier runs two companies, two properties across Belgium and Spain, and a growing set of automation tools. His current AI assistant (an open-source tool called Openclaw) runs 6 cron jobs on a Mac Mini — monitoring email, WhatsApp, system health, and delivering daily briefings. After two weeks in production, three problems emerged: poor observability, too much manual configuration, and the best AI models being economically out of reach for an always-on agent.

The decision was made: replace Openclaw with purpose-built Go microservices. Three design documents were written and approved:

- **Service conventions** — the contract every service follows (config, logging, health checks, audit trail)
- **Migration strategy** — which Openclaw cron to replace first and why
- **Architecture research** — the analysis that led to the Go decision

These documents existed in the `chepibe` repository (Xavier's private instance repo). What did not exist yet: the Go codebase, the development environment, CI, or a backlog of work items derived from those designs.

### What the Agilar AI SDLC provided

Xavier had recently completed PBI #138: designing the Agilar AI SDLC methodology itself. The methodology (now part of `agilar-coder`) contained:

- **SCRUM.md** — the Agile framework adapted for AI-enabled teams (Kanban for solo, Scrum for multi-human)
- **DEVOPS.md** — pipeline definition, quality gates, branching strategies
- **TOOLCHAIN.md** — recommended tools and what degrades without them
- **14 skills** — executable process definitions for TDD, debugging, brainstorming, code review, verification, BDD, and more
- **A scaffold wizard** — a bash script that generates project configuration from templates
- **Stack-specific references** — Go, Node.js, Python, Elixir configs for linting, testing, CI
- **A CLAUDE.md template** — the working agreements file that tells the AI agent how to behave

### The task

PBI #137: "Sprint Zero — dev environment, CI, and product backlog for chepibe-core." Take the approved designs and create everything needed so the next session can pull PBI #148 (the WhatsApp auto-responder) and start writing Go code with tests.

---

## What was done

### Step 1: Development environment

Go 1.26.0 and golangci-lint 2.10.1 were installed on the development machine via Homebrew. The production server (M3) already had Go installed. This took under a minute.

**What the methodology says:** DEVOPS.md requires that the dev environment can run the full test suite locally. TOOLCHAIN.md lists linters as "recommended" and says to treat warnings as errors. The Go stack reference (`templates/stacks/go.md`) specifies golangci-lint with a recommended `.golangci.yml` configuration.

### Step 2: Repository creation

`chepibe-core` was created as a private GitHub repository and cloned locally. The methodology's engine-vs-instance separation (defined in the service conventions design doc) dictated this: the Go source code lives in `chepibe-core` (portable, future open source), while Xavier's personal config, secrets, and data stay in the existing `chepibe` repo.

### Step 3: Methodology scaffolding

This is where the Agilar AI SDLC framework did most of its work.

The scaffold wizard defines a standard project setup: CLAUDE.md (working agreements), `.claude/skills/` (executable skill implementations for Claude Code), `docs/stack-reference.md` (language-specific config), and `docs/skills/` (canonical skill definitions as reference).

For chepibe-core, the configuration was:

| Parameter | Value |
|-----------|-------|
| Team mode | Solo (1 human + 1 agent) |
| Stack | Go |
| Tool | Claude Code |
| Code review | Enabled |
| Branching | Trunk-based (all commits on main) |

The scaffold produced:

**CLAUDE.md** — The working agreements file. This is the single most important output. It tells the AI agent:

- Five non-negotiable working agreements (TDD, root cause debugging, verification with evidence, brainstorming before design, code review)
- Definition of Ready (small enough, acceptance criteria, design approved, no blockers)
- Definition of Done (criteria met, tests pass, verified with evidence, code reviewed, docs updated)
- Exact commands: `make build`, `make test`, `make lint`
- Architecture context: CHEPIBE_ROOT convention, engine-vs-instance split, service naming
- The PBI-first rule: any new work gets a PBI before starting

The CLAUDE.md was then customized with chepibe-core-specific notes: references to the design documents, the `pkg/` shared library structure, service conventions, and migration order. The template gave the structure; the project context filled the details.

**14 skill files** (`.claude/skills/*.md`) — copied from `implementations/claude-code/`. These are executable process definitions:

| Skill | What it enforces |
|-------|-----------------|
| `tdd` | Red-green-refactor cycle. Write the test first, watch it fail, make it pass, refactor. Every time. |
| `debugging` | Four-phase root cause investigation before any fix attempt |
| `verification` | Fresh evidence (test output, screenshots) before any completion claim |
| `brainstorming` | Explore alternatives before committing to a design |
| `code-review` | Two-stage review: spec compliance first, then code quality |
| `bdd` | Acceptance tests as executable Gherkin specifications |
| `sprint-planning` | Break PBIs into executable implementation tasks |
| `executing-plans` | Execute plans with checkpoints and review gates |
| `scrum-master` | Process conscience — nudges when working agreements are at risk |
| `po-coach` | Product ownership guidance during refinement |
| `facilitator` | Scrum event facilitation |
| `parallel-agents` | Dispatch independent tasks to parallel agents |
| `subagent-driven` | Orchestrate sub-agents for complex tasks |
| `git-worktrees` | Isolated feature development in git worktrees |

These are not documentation. They are instructions the AI agent reads at the start of every relevant task. When the agent is about to write code, the TDD skill tells it exactly how to proceed. When a bug is found, the debugging skill prescribes a four-phase investigation. The agent does not choose whether to follow them — the CLAUDE.md working agreements make them mandatory.

**Stack reference** (`docs/stack-reference.md`) — Go-specific configuration: test commands, lint setup, build commands, BDD framework (godog), pre-commit hook template.

### Step 4: Go module scaffold

The Go project structure was created:

```
chepibe-core/
├── go.mod                    # github.com/xquesada/chepibe-core
├── Makefile                  # build, test, lint, clean
├── .golangci.yml             # linter config (v2 format)
├── .gitignore                # bin/, test artifacts, IDE files
├── cmd/chepibe-ctl/main.go   # minimal placeholder (so go build works)
├── CLAUDE.md
├── README.md
├── .claude/skills/           # 14 skill files
├── .github/workflows/ci.yml
└── docs/
    ├── stack-reference.md
    └── testing-strategy.md
```

**Key decisions guided by the methodology:**

- **Makefile over scripts** — TOOLCHAIN.md recommends standardized targets. `make build`, `make test`, `make lint` are the commands in CLAUDE.md. The agent will use these exact commands in every session.
- **Minimal placeholder** — only `cmd/chepibe-ctl/main.go` was created (a 5-line program that prints a version string). The plan explicitly said "do NOT create cmd/ or pkg/ subdirectories yet — those get their own PBIs." The PBI-first rule applies even to directory structure.
- **golangci-lint v2 config** — the initial config used v1 syntax (`linters-settings` as a top-level key). CI caught this — the config verification step failed. The fix was straightforward: move settings under `linters.settings`. This is exactly the kind of thing CI exists to catch.

### Step 5: CI pipeline

GitHub Actions workflow (`.github/workflows/ci.yml`):

```yaml
- Trigger: push to main, PRs
- Steps: checkout → setup-go → golangci-lint → test → build
```

**What the methodology says:** DEVOPS.md defines quality gates. Gate 1 (Test Gate): all tests pass before commit. The CI pipeline encodes this: test failure blocks merge. The Go stack reference provides the recommended CI configuration. TOOLCHAIN.md calls CI "recommended" and notes that without it, "working agreements become trust-based."

**What actually happened:** The first CI run failed. The golangci-lint v2 config format was different from v1 (a top-level `linters-settings` key that v2 doesn't allow). The fix was a one-line change. The second run passed in 28 seconds: lint clean, tests pass (trivially — no test files yet), build succeeds.

This failure-then-fix sequence is worth noting. It happened in Sprint Zero, where the cost was near-zero. Had it happened after 50 PBIs with real code, it would have been more disruptive. CI catches config issues early.

### Step 6: Testing strategy

A `docs/testing-strategy.md` was written defining four test categories:

| Category | Build tag | What it tests | Requirements |
|----------|-----------|---------------|-------------|
| Unit | (none) | Logic, functions | Nothing |
| Integration | `integration` | Database, config loading | SQLite in-memory |
| LLM | `llm` | Prompt-to-response flows | Ollama running locally |
| BDD | `bdd` | Service-level acceptance | godog framework |

**Why this matters:** The TDD skill will be active from the very first PBI. The testing strategy tells the agent which test category to use for each scenario. Unit tests run on every `go test ./...`. Integration and LLM tests are gated behind build tags so the fast path stays fast.

### Step 7: Instance directory

The `chepibe` (instance) repo was extended with the directories the Go services will need:

- `config/chepibe.yaml` — global config (alert channels, LLM provider, database path)
- `prompts/` — LLM prompt templates (empty, ready for first service)
- `deploy/` — launchd/systemd unit files (empty, ready for first service)
- `bin/` and `data/` added to `.gitignore`

This separation was defined in the service conventions design doc. The Agilar methodology didn't prescribe this specific split, but the PBI-first rule influenced it: the instance directory setup was its own step, not mixed into the Go scaffold work.

### Step 8: Commit, push, verify

Two commits to `chepibe-core`:

1. "Sprint Zero: Go module scaffold, CI, methodology" — 24 files, the full scaffold
2. "fix: golangci-lint v2 config format" — 1 file, the CI fix

Two commits to `chepibe`:

1. "chepibe-core: add instance directory structure" — config, prompts, deploy directories
2. "chepibe-core: gitignore bin/ and data/ for Go services"

CI ran green on the second push. Build time: 28 seconds.

### Step 9: Product backlog

The three design documents were decomposed into 15 Product Backlog Items, all created via the backlog API (the methodology's PBI-first rule: "any new work gets a PBI before starting").

**Foundation packages (7 PBIs):**

| # | PBI | Purpose |
|---|-----|---------|
| 141 | pkg/log | slog wrapper with service name and format |
| 142 | pkg/config | Config loading and secrets lookup |
| 143 | pkg/db | SQLite WAL mode and migrations |
| 144 | pkg/audit | LLM call and service action logging |
| 145 | pkg/llm | OpenAI-compatible HTTP client |
| 146 | pkg/alert | State-transition alerts with Telegram |
| 147 | pkg/health | /healthz HTTP server for daemons |

**Services (4 PBIs, in migration order):**

| # | PBI | Type |
|---|-----|------|
| 148 | chepibe-whatsapp | Daemon — WhatsApp business auto-responder |
| 149 | chepibe-gmail | Scheduled — email triage |
| 150 | chepibe-heartbeat | Scheduled — status checks |
| 151 | chepibe-agent | Daemon — multi-channel conversational agent |

**Tooling and deploy (3 PBIs):**

| # | PBI |
|---|-----|
| 152 | chepibe-ctl — CLI management tool |
| 153 | chepibe-ctl — web setup wizard |
| 154 | Deploy templates — launchd and systemd |

**Migration (1 PBI):**

| # | PBI |
|---|-----|
| 155 | Openclaw data extraction and cron disable sequence |

Each PBI was created with:
- A descriptive title
- Notes with context and references to the relevant design document section
- A checklist of discrete action steps (the methodology's checklist convention: "action steps go in the checklist, context stays in notes")
- Status `backlog` — none are `ready` yet, because they haven't been refined with acceptance criteria

PBIs #136 (repo creation) and #137 (Sprint Zero itself) were marked `done`.

---

## How the Agilar framework shaped the work

### What the methodology prescribed

**The CLAUDE.md template** gave the project its working agreements from minute one. Before any Go code existed, the agent knew: TDD is mandatory, verification requires evidence, root cause before fix, brainstorm before design. These aren't aspirational goals — they're rules the agent will enforce starting with the very first PBI.

**The scaffold wizard's structure** standardized the project setup. Skills directory, stack reference, CLAUDE.md, docs — every Agilar project starts the same way. A developer picking up chepibe-core for the first time will find the same file layout they'd find in any other Agilar-scaffolded project.

**SCRUM.md's PBI-first rule** drove the backlog creation. The design documents contained all the information about what to build, but they weren't actionable units of work. Breaking them into PBIs with checklists made each piece pullable: small enough for one session, concrete enough to know when it's done.

**DEVOPS.md's quality gates** determined the CI pipeline. Test gate (all tests pass), review gate (code reviewed), verification gate (evidence before claims). The CI workflow encodes the first gate. The others are encoded in the CLAUDE.md working agreements and enforced by the agent's skills.

**The Go stack reference** provided the linter config, test commands, and BDD framework recommendation. This saved research time — the config was tested and ready to use.

### What the methodology left to the project

The methodology is deliberately silent on architecture decisions. It doesn't tell you to use SQLite or Postgres, microservices or monolith, Go or Python. Those decisions came from the design documents that Xavier and his agent wrote during PBI #130 (architecture research). The methodology provided the process for making those decisions (brainstorming skill, sprint-planning skill), but the decisions themselves belong to the Product Owner.

The service conventions (config layout, logging format, health checks, audit trail) are project-specific. The methodology says "have structured logs" (DEVOPS.md, observability section) and "have an audit trail" (DEVOPS.md, agent observability). The service conventions doc defined exactly how.

### What would have been different without the methodology

Without the scaffold, Sprint Zero would have required making dozens of small decisions: What goes in the CLAUDE.md? What skills should the agent have? What linter config? What CI steps? What's the Definition of Done? Each decision is individually small, but collectively they consume significant time and cognitive load.

The scaffold collapsed those decisions into a single configuration step: Solo mode, Go stack, Claude Code, code review enabled. The template filled in the rest. The customization (architecture notes, service conventions references, pkg/ library documentation) was additive — building on a solid default rather than starting from scratch.

Without the PBI-first rule, the backlog creation might have been deferred. "We'll create tickets as we go" is a common pattern that leads to work without acceptance criteria, scope creep, and no clear definition of done. The 15 PBIs created during Sprint Zero mean that when the next session starts, the first action is: pull PBI #148, refine it with acceptance criteria, and start building.

---

## Output

### What exists after Sprint Zero

| Artifact | Location | State |
|----------|----------|-------|
| GitHub repo | `xquesada/chepibe-core` (private) | Created, 2 commits |
| Go module | `go.mod` (github.com/xquesada/chepibe-core) | Compiles, Go 1.26 |
| CI pipeline | GitHub Actions | Green (lint + test + build in 28s) |
| Working agreements | `CLAUDE.md` | 5 agreements, DoR, DoD, all skills listed |
| AI agent skills | `.claude/skills/` | 14 executable skill files |
| Linter | `.golangci.yml` | golangci-lint v2, 5 linters enabled |
| Build system | `Makefile` | build, test, lint, clean targets |
| Testing strategy | `docs/testing-strategy.md` | 4 categories with build tags |
| Stack reference | `docs/stack-reference.md` | Go commands, BDD, pre-commit hook |
| Instance config | `chepibe/config/chepibe.yaml` | Global config template |
| Instance dirs | `chepibe/prompts/`, `chepibe/deploy/` | Empty, ready for services |
| Product backlog | 15 PBIs (#141-#155) | All `backlog` status, with checklists |

### What does NOT exist yet

- No `pkg/` packages — each one is a separate PBI
- No `cmd/` services beyond the placeholder — each one is a separate PBI
- No production code — Sprint Zero is infrastructure, not features
- No tests — there's nothing to test yet (TDD will produce tests alongside the first real code)
- No acceptance criteria on the new PBIs — refinement happens when each PBI is pulled

### What happens next

The Kanban workflow from SCRUM.md takes over:

1. Pull PBI #148 (chepibe-whatsapp) from the top of the backlog
2. Refine it: write acceptance criteria, confirm design, ensure it meets Definition of Ready
3. Build it following the skills: TDD (write test, watch fail, make pass), verification (fresh evidence), code review
4. Mark it done when it meets Definition of Done
5. Pull the next one

The foundation packages (#141-#147) will be built as needed by the first service, not in isolation. The design documents say "extract common patterns after building 2-3 real services, not before." `pkg/log` and `pkg/config` will likely emerge from chepibe-whatsapp's needs. The PBIs exist so the work is tracked, but the order is driven by what the current service requires.

---

## Observations

### What worked well

**The scaffold eliminated decision fatigue.** Sprint Zero is inherently a "lots of small decisions" phase. The Agilar scaffold made most of them non-decisions: the template has opinions, use them, customize what's specific to your project.

**CI caught a real issue.** The golangci-lint v2 config format was wrong. Locally it worked because the tool was lenient. CI's config verification step caught it. Cost of fixing it in Sprint Zero: 2 minutes and one commit. Cost of discovering it later with a full codebase: debugging CI failures while trying to ship a feature.

**The PBI-first rule forced backlog discipline.** Creating 15 PBIs from the design documents was a deliberate act of decomposition. Each PBI has a title, notes with context, and a checklist of action steps. This isn't busywork — it's the translation from "design document that describes a system" to "ordered list of things to build, each small enough for one session."

**Skills as executable processes, not guidelines.** The 14 skill files in `.claude/skills/` are not documentation for humans to read. They are instructions the AI agent loads and follows. When the next session starts and the agent begins implementing chepibe-whatsapp, the TDD skill will tell it to write a failing test before any production code. The verification skill will require fresh test output before claiming the PBI is done. The agent doesn't choose whether to follow these — the working agreements in CLAUDE.md make them non-negotiable.

### What the methodology does not do

It does not make architecture decisions. The Go choice, the microservices pattern, the SQLite persistence, the service naming conventions — all came from the product vision and design discussions, not from the methodology. The methodology provided the process for having those discussions (brainstorming skill, sprint-planning skill), but the decisions belong to the human.

It does not write code. Sprint Zero produced zero production code. The methodology is explicit about this: the PBI-first rule says work gets a PBI before starting. The Definition of Ready says acceptance criteria must be written before a PBI is pulled. Sprint Zero created the conditions for production code to be written correctly — in the next session, following TDD, with CI catching regressions, and a clear backlog of what to build.

It does not guarantee success. What it guarantees is discipline. Every PBI will go through TDD. Every completion claim will have evidence. Every bug fix will investigate root cause. These practices reduce the probability of certain failure modes (untested code, unverified claims, symptom-chasing). They don't eliminate the possibility of building the wrong thing — that's the Product Owner's responsibility.

---

## Post-Sprint Zero: The PO/Dev Boundary

An important process learning emerged during PBI refinement, after Sprint Zero was complete.

### The problem

PBI #148 (the first real service to build) was refined in the `chepibe` repo — the instance repo where the backlog, business rules, and design documents live. The natural next step was to start building it. But building it from the `chepibe` repo would mean the agent never loads the `chepibe-core` CLAUDE.md, never activates the 14 skills, never enforces the working agreements. The entire Sprint Zero scaffold would be inert.

### The insight

The methodology has a natural boundary between product ownership and development that maps to separate workspaces:

| Layer | Repo | Role | Analogy |
|-------|------|------|---------|
| Product ownership | `chepibe` (instance) | Define, refine, prioritize PBIs. Write acceptance criteria. Approve designs. | Jira, PO Companion |
| Development | `chepibe-core` (engine) | Build, test, verify. Skills active. Working agreements enforced. | The IDE + dev environment |

The PO refines the PBI in the backlog system. The developer opens a new session in the project repo, queries the backlog for the current PBI, and builds it — with TDD, brainstorming, verification, all the skills loaded and active.

### What was added

The `chepibe-core` CLAUDE.md was updated with a "Product Backlog" section that tells the agent:
- Where PBIs live (the backlog API in the instance repo)
- How to query them at session start
- To read acceptance criteria and checklist before starting work
- To mark the PBI `in_progress` when beginning

This mirrors how a developer opens Jira to read the ticket before writing code — except the agent does it programmatically via API.

### Implication for the framework

The Agilar AI SDLC scaffold wizard originally generated only the development environment (CLAUDE.md, skills, CI). It did not ask where requirements live or how the agent accesses them. This gap was identified during Sprint Zero and addressed in two passes:

**Pass 1 (same day):** The scaffold was updated with a Step 6 that asks "Where do you manage requirements?" and generates the CLAUDE.md `## Product Backlog` section with the right configuration. This tells the agent WHERE the backlog is.

**Pass 2 (PBI #156):** The po-coach skill was updated with a **backlog adapter pattern** — concrete instructions that tell the agent HOW to interact with the configured backlog tool. Six logical operations (list ready PBIs, read details, mark in_progress, mark done, read acceptance criteria, update checklist) are mapped to tool-specific commands for each backlog option:

| Backlog tool | Access mechanism |
|---|---|
| REST API (PO Companion / Chepibe) | `curl` commands via Bash |
| YAML file in repo | Read/Edit tools directly |
| GitHub Issues | `gh` CLI |
| MCP-based tools (Jira, Miro, Linear) | MCP server tools |

The adapter is instructions, not code. It lives in the po-coach skill because that skill bridges the PO and dev layers. The scrum-master and verification skills cross-reference it as the single source of truth for backlog access.

This separation is not unique to AI-assisted development. It's the same PO/developer boundary that exists in any Scrum team. But in AI-assisted development, it has a concrete technical expression: the CLAUDE.md that governs agent behavior must include a link to the backlog system, and the skills must know how to use it — or the agent works blind.

### First launch from the dev layer

After completing the backlog adapter (PBI #156) and syncing the updated skills to chepibe-core, we launched Claude Code from `~/projects/chepibe-core` for the first time — with no prior context, no warm-up, just the scaffold and a prompt: "Do a full sanity check. Are your working agreements clear? Is everything coherent?"

The agent self-oriented in under a minute:

| Check | Result |
|-------|--------|
| Go toolchain | 1.26.0, golangci-lint 2.10.1 |
| Build / Test / Lint | All green |
| Git + CI | Clean, GitHub Actions configured |
| Backlog API | Reachable, 17 chepibe-core PBIs loaded |
| Working agreements | Understood and listed all 6 |
| Next PBI | #148 identified as ready, offered to start brainstorming |

The agent found the `## Product Backlog` section in CLAUDE.md, queried the REST API, filtered by `epic == "chepibe-core"`, and presented the backlog state — without being told how. The po-coach adapter did its job.

One issue surfaced: a `wake-up` skill (designed for the chepibe instance repo) was installed globally and fired in chepibe-core where it didn't belong. It tried to SSH to the home server and check agent coordination files that don't exist in chepibe-core. The fix took 30 seconds — move the skill from global scope (`~/.claude/skills/`) to project scope (`~/.claude/projects/.../skills/`). This is a general lesson: **skills that reference project-specific infrastructure must be scoped to that project, not installed globally.**

The framework's end-to-end flow is now validated:

```
scaffold → skills installed → CLAUDE.md configured → backlog adapter active
    → agent self-orients → queries backlog → finds next PBI → ready to build
```

No manual explanation needed. No "here's how to find your work." The agent reads its own configuration and follows its own skills. This is what "opinionated by default" means in practice — the methodology made the decisions during setup so the agent doesn't need the human to repeat them every session.
