# Agilar AI SDLC

A methodology for AI-enabled Agile Software Development.

Humans design. AI executes. Quality is non-negotiable.

## Three Pillars

### 1. Skills as Executable Process

Development practices are skill files that AI agents load and follow. TDD isn't a guideline — it's a skill file the agent enforces. Debugging isn't advice — it's a four-phase process the agent executes step by step.

Skills live in `skills/`. Each one defines a process with iron laws, checklists, and red flags. Agents don't interpret them — they follow them.

### 2. Human Designs, AI Executes

Clear role contract:

| Role | Human | AI Agent |
|------|-------|----------|
| **Brainstorm** | Drives the conversation | Proposes approaches, asks questions |
| **Decide** | Makes all decisions | Presents options with trade-offs |
| **Design** | Approves architecture | Explores codebase, drafts designs |
| **Implement** | Reviews code | Writes code following TDD |
| **Verify** | Confirms done | Runs tests, provides evidence |

The human is always the Product Owner. The AI is a skilled developer who follows process rigorously.

### 3. Iron Laws

Non-negotiable quality gates baked into skills:

- **No production code without a failing test first** (TDD)
- **No fix without root cause investigation** (Debugging)
- **No completion claim without fresh verification evidence** (Verification)
- **No design without exploring alternatives** (Brainstorming)
- **No merge without code review** (Code Review)

Iron laws are not guidelines. Violating the letter is violating the spirit.

## Team Modes

The methodology scales across three configurations:

| Mode | People | Agents | Process | Branching |
|------|--------|--------|---------|-----------|
| **Solo** | 1 human (PO + Dev) | 1 agent (spawns sub-agents) | Kanban | Trunk (main) |
| **Multi-agent** | 1 human (PO + Dev) | Agent team (orchestrator + workers) | Kanban | Worktrees + main |
| **Multi-human** | PO + Developer(s) | Agent(s) per developer | Scrum | Feature branches + main |

Solo and multi-agent use Kanban: pull next ready PBI, no sprint ceremonies. Multi-human uses full Scrum with sprint planning, review, and retrospective.

## Getting Started

### Quick Start

1. Read this README
2. Read [SCRUM.md](SCRUM.md) for the Agile framework
3. Read [DEVOPS.md](DEVOPS.md) for environments and quality gates
4. Read [TOOLCHAIN.md](TOOLCHAIN.md) for recommended tools
5. Browse `skills/` for the development practices

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

## Structure

```
agilar-ai-sdlc/
├── README.md                    # This file
├── SCRUM.md                     # Agile framework: roles, artifacts, events
├── DEVOPS.md                    # Pipeline: environments, quality gates, CI/CD
├── TOOLCHAIN.md                 # Recommended tools + alternatives
├── skills/                      # Canonical skill definitions (tool-agnostic)
│   ├── brainstorming/SKILL.md
│   ├── tdd/SKILL.md
│   ├── writing-plans/SKILL.md
│   ├── executing-plans/SKILL.md
│   ├── debugging/SKILL.md
│   ├── code-review/SKILL.md
│   ├── verification/SKILL.md
│   ├── bdd/SKILL.md
│   ├── subagent-driven/SKILL.md
│   ├── parallel-agents/SKILL.md
│   ├── git-worktrees/SKILL.md
│   ├── scrum-master/SKILL.md
│   ├── po-coach/SKILL.md
│   └── facilitator/SKILL.md
├── implementations/             # Tool-specific implementations
│   ├── claude-code/
│   ├── cursor/
│   └── generic/
├── templates/
│   ├── CLAUDE.md.tmpl
│   └── stacks/
├── scaffold                     # Setup wizard
└── examples/
```

## Philosophy

This methodology is extracted from practice — building real products with AI agents — not invented at a desk. Every skill, every iron law, every process step exists because we learned the hard way what happens without it.

It's opinionated. It's rigid where it needs to be (iron laws) and flexible where it can be (team modes, toolchain). The goal is not to prescribe every detail but to encode the discipline that makes AI-assisted development reliable.

The methodology IS the repo. You learn by reading the skills and using the tools. There is no separate guide.

## License

Proprietary. Copyright Agilar.
