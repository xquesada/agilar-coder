# Agilar Agentic Coding

The **Agilar Agentic Coding methodology** — Agilar's method for AI-enabled Agile Software Development.

The terms "methodology", "method", and "framework" all refer to the same thing: the Agilar Agentic Coding methodology, implemented by the `agilar-coder` CLI.

Humans design. AI executes. Quality is non-negotiable.

## Three Pillars

### 1. Human Designs, AI Executes

Clear role contract between humans and AI agents:

| Role | Human | AI Agent |
|------|-------|----------|
| **Brainstorm** | Drives the conversation | Proposes approaches, asks questions |
| **Decide** | Makes all decisions | Presents options with trade-offs |
| **Design** | Approves architecture | Explores codebase, drafts designs |
| **Implement** | Reviews code | Writes code following TDD |
| **Verify** | Confirms done | Runs tests, provides evidence |

The human is always the Product Owner. The AI is a skilled developer who follows process rigorously. Decisions always belong to the human. Process discipline always belongs to the agent.

### 2. Working Agreements

Quality gates the team commits to:

- **No production code without a failing test first** (TDD)
- **No fix without root cause investigation** (Debugging)
- **No completion claim without fresh verification evidence** (Verification)
- **No design without exploring alternatives** (Brainstorming)
- **No merge without code review** (Code Review — optional for non-technical solo users)

Working agreements are not guidelines. The team agrees to follow them, and the agent enforces them. Violating the letter is violating the spirit.

### 3. Opinionated by Default

This methodology makes choices for you. It doesn't say "pick an Agile framework" — it says use Scrum or Kanban, depending on your team mode, and here are the exact events, artifacts, and roles. It doesn't say "have some tests" — it says TDD with red-green-refactor, and here's the exact process with verification at every step.

The opinions come from Agilar's experience building real products with AI agents. They're encoded in three places:

- **[SCRUM.md](SCRUM.md)** — How you manage work: Scrum for multi-human teams, Kanban for solo and multi-agent
- **[DEVOPS.md](DEVOPS.md)** — How code moves to production: environments, quality gates, branching, CI/CD
- **Skills** (`skills/`) — How you develop: 17 skills covering TDD, debugging, brainstorming, code review (performing, requesting, receiving), verification, multi-agent orchestration, and more — each one a concrete, executable process

You can adapt the opinions to your context. But the defaults work. Start with them and change only what you need to.

## Team Modes

The methodology scales across three configurations:

| Mode | People | Agents | Process | Branching |
|------|--------|--------|---------|-----------|
| **Solo** | 1 human (PO + Dev) | 1 agent (spawns sub-agents) | Kanban | Trunk (main) |
| **Multi-agent** | 1 human (PO + Dev) | Agent team (orchestrator + workers) | Kanban | Worktrees + main |
| **Multi-human** | PO + Developer(s) | Agent(s) per developer | Scrum | Feature branches + main |

Solo and multi-agent use Kanban: pull next ready PBI, no sprint ceremonies. Multi-human uses full Scrum with sprint planning, review, and retrospective.

## The CLI (`agilar-coder`)

A bash script that runs Claude Code unattended, processing Product Backlog Items from a markdown file one at a time.

```bash
./agilar-coder backlog.md          # Process all PBIs until done
./agilar-coder backlog.md 3        # Process exactly 3 PBIs
./agilar-coder --debug backlog.md  # Show command without executing
```

Config: `~/.agilar-coder/config.conf` (created on first run). Full specification: [SPEC.md](SPEC.md).

## Getting Started

### Quick Start

1. Read this README
2. Read [SCRUM.md](SCRUM.md) for the Agile framework
3. Read [DEVOPS.md](DEVOPS.md) for environments and quality gates
4. Read [TOOLCHAIN.md](TOOLCHAIN.md) for recommended tools
5. Browse `skills/` for the development practices
6. Try the CLI: `./agilar-coder --help`

### Scaffolding a New Project

Run the scaffold wizard to set up a new project with the methodology:

```bash
./scaffold
```

The wizard asks about your project, team mode, and toolchain, then generates all configuration files tailored to your answers.

### Using Skills

Skills are in `skills/`. Each has a `SKILL.md` with the tool-agnostic process definition. Tool-specific implementations live in `implementations/`:

- `implementations/claude-code/` — `.claude/skills/` format for Claude Code
- `implementations/cursor/` — `.cursorrules` format for Cursor (future)
- `implementations/generic/` — Plain markdown for any tool

### Upgrading

When a new version of agilar-coder is available, update your project:

```bash
agilar-coder upgrade /path/to/your/project
```

The upgrade command will:
1. Update skills (with diff review for modified ones)
2. Detect and offer to remove deprecated skills
3. Show what changed between your installed version and the new one (from `CHANGELOG.md`)
4. Print any manual migration steps required for breaking changes
5. Bump your project's version marker

Check what version you're on with `agilar-coder status`.

See [CHANGELOG.md](CHANGELOG.md) for the full version history.

## Structure

```
agilar-coder/
├── agilar-coder                 # CLI bash script
├── SPEC.md                      # CLI specification
├── VERSION                      # Semver version
├── README.md                    # This file
├── SCRUM.md                     # Agile framework: roles, artifacts, events
├── DEVOPS.md                    # Pipeline: environments, quality gates, CI/CD
├── TOOLCHAIN.md                 # Recommended tools + alternatives
├── scaffold                     # Project setup wizard
├── skills/                      # Canonical skill definitions (tool-agnostic)
│   ├── brainstorming/SKILL.md
│   ├── tdd/SKILL.md
│   ├── sprint-planning/SKILL.md
│   ├── executing-plans/SKILL.md
│   ├── debugging/SKILL.md
│   ├── code-review/SKILL.md
│   ├── requesting-code-review/SKILL.md
│   ├── receiving-code-review/SKILL.md
│   ├── verification/SKILL.md
│   ├── bdd/SKILL.md
│   ├── subagent-driven/SKILL.md
│   ├── parallel-agents/SKILL.md
│   ├── git-worktrees/SKILL.md
│   ├── finishing-a-development-branch/SKILL.md
│   ├── scrum-master/SKILL.md
│   ├── po-coach/SKILL.md
│   └── facilitator/SKILL.md
├── implementations/             # Tool-specific implementations
│   ├── claude-code/
│   ├── cursor/
│   └── generic/
├── templates/
│   ├── CLAUDE.md.tmpl
│   ├── agents/                  # Generic agent role templates
│   │   ├── code-reviewer.md
│   │   └── ci-checker.md
│   └── stacks/                  # Stack-specific templates + agent roles
└── examples/
    ├── solo-go/
    ├── team-node/
    └── multi-agent-web/         # Multi-agent + Node.js (with agent definitions)
```

## Philosophy

This methodology is extracted from practice — building real products with AI agents — not invented at a desk. Every skill, every working agreement, every process step exists because we learned the hard way what happens without it.

It's opinionated. It's rigid where it needs to be (working agreements) and flexible where it can be (team modes, toolchain). The goal is not to prescribe every detail but to encode the discipline that makes AI-assisted development reliable.

The methodology IS the repo. You learn by reading the skills and using the tools. There is no separate guide.

## License

Proprietary. Copyright Agilar.
