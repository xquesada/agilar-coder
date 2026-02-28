# Agilar Toolchain

Recommended tools, what they do for the Agilar AI SDLC methodology, and what degrades without them.

## Two Layers

This methodology separates **principles** from **tools** deliberately.

**Layer 1 -- Principles (tool-agnostic).** SCRUM.md, DEVOPS.md, skill intents, working agreements, team modes. These work with any AI coding assistant, any CI/CD system, any hosting platform. A team using Cursor instead of Claude Code, GitLab instead of GitHub, Jenkins instead of GitHub Actions still follows the same methodology. The skills define *what* to do and *why*. The toolchain defines *how* to automate it.

**Layer 2 -- Toolchain (recommended, not required).** Specific tool implementations that make the methodology easier to follow. Claude Code skills make process steps executable. Entire captures the audit trail automatically. GitHub Actions enforces working agreements in CI. These tools reduce friction and catch mistakes -- but the methodology works without any of them. You just do more manually.

The line between layers is simple: if removing a tool changes *what* you do, it's in the wrong layer. Removing a tool should only change *how much effort* it takes.

## Tools at a Glance

| Tool | Role in methodology | Status | Without it |
|------|-------------------|--------|------------|
| **Claude Code** | AI agent, skills system, agent teams, sub-agents | Recommended | Skills become guidelines instead of executable. Any AI assistant works but with less automation. |
| **Entire** | Git-native audit trail for AI agent work | Recommended | Lose automatic transcript/change capture. Manual audit logging still works. |
| **Git** | Version control, branching, worktrees | Required | Methodology assumes git. |
| **Git worktrees** | Agent isolation in multi-agent mode | Optional | Only needed for multi-agent. Alternative: feature branches. |
| **GitHub / GitLab** | Code hosting, PR workflow, CI/CD | Recommended | Any git hosting works. Multi-human mode needs PR workflow. |
| **GitHub Actions / GitLab CI** | Automated quality gates | Recommended | Manual gate enforcement. CI automates working agreements. |
| **Testing frameworks** | TDD and verification evidence | Required (any) | TDD skill cannot function without a test runner. |
| **ATDD/BDD frameworks** | Acceptance test automation (Gherkin) | Strongly recommended | Acceptance criteria remain manual. Lose executable specifications. |
| **Linters / formatters** | Automated code quality | Recommended | Manual code style enforcement. One less CI gate. |
| **Docker / containers** | Environment reproducibility | Recommended | Manual environment parity. Staging drift becomes likely. |
| **Infrastructure as Code** | Staging/production parity | Optional | Manual infrastructure management. Acceptable for small deployments. |

---

## Git

### What it does for the methodology

Everything. The methodology assumes git as the version control system. Branching strategies (trunk-based for solo/multi-agent, feature branches for multi-human) are defined in SCRUM.md. The TDD skill assumes commits at green. The code-review skill assumes diffs. DEVOPS.md assumes git-triggered pipelines. The audit trail stores transcripts as git commits.

Git is the only tool in the "Required" column because it is woven into every other practice.

### How the methodology degrades without it

It doesn't degrade. It stops working. Branching strategies, commit discipline, worktree isolation, CI triggers, audit trail storage -- all assume git.

### Alternatives

None within this methodology. If you use Mercurial, SVN, or Perforce, the principles still apply but the toolchain sections need rewriting.

---

## Claude Code

### What it does for the methodology

Claude Code is the reference AI coding agent. Three capabilities matter:

1. **Skills system.** Skills in `skills/` define tool-agnostic processes. The `implementations/claude-code/` directory translates them into `.claude/skills/` files that Claude Code loads and follows directly. The TDD skill becomes an executable process the agent enforces, not a guideline it might follow. Same for debugging, brainstorming, code review, verification -- every skill in `skills/`.

2. **Sub-agents.** The `skills/subagent-driven/` skill uses Claude Code's ability to spawn sub-agents for isolated tasks. One agent writes tests, another implements, a third reviews. Each sub-agent gets a focused prompt and returns structured results. This enables the multi-agent team mode described in SCRUM.md.

3. **Worktree support.** Claude Code can work in git worktrees natively. Combined with the `skills/parallel-agents/` skill, multiple agent instances work on separate PBIs simultaneously without stepping on each other.

### How the methodology degrades without it

Skills become documents instead of executables. A developer using a different AI assistant reads the `skills/` directory and follows the process manually -- or instructs their assistant to follow it. The working agreements still apply, but enforcement shifts from automatic to human.

Sub-agent orchestration disappears. Multi-agent mode still works (multiple terminal sessions, each with its own AI assistant and worktree) but without programmatic coordination.

The methodology still works. It's just more manual.

### Alternatives

- **Cursor** -- Rules files (`.cursorrules`) can encode skill processes. Less structured than Claude Code skills but functional. The `implementations/cursor/` directory will provide these translations.
- **GitHub Copilot** -- Chat mode can follow process instructions from context files. No skills system, so everything is prompt-based.
- **Codex (OpenAI)** -- Task-based agent with sandbox execution. Can follow process instructions. No native skills system.
- **Antigravity** -- AI coding agent. Can read skill files as context. Different automation model from Claude Code.
- **Any AI assistant** -- The `implementations/generic/` directory provides plain markdown that any tool can use as instructions.

---

## Entire

### What it does for the methodology

Entire captures a git-native audit trail of AI agent work. Every agent session produces a transcript: what was asked, what the agent did, what files changed, what tests ran. Entire stores these as structured data in the git history alongside the code changes.

This serves two purposes:

1. **Accountability.** When something breaks, you can trace back to which agent session made the change, what the human asked for, and what the agent's reasoning was. The verification skill (`skills/verification/`) requires evidence of test results -- Entire captures that evidence automatically.

2. **Learning.** Retrospectives (SCRUM.md) use session transcripts to identify what went well and what didn't. Without transcripts, retrospectives rely on memory.

### How the methodology degrades without it

You lose automatic capture. The verification skill still requires evidence, but the developer has to save it manually (screenshots, log snippets, copy-pasted test output). The code-review skill still works, but reviewers see only the diff, not the reasoning behind it.

For solo mode, the degradation is minor -- you were there, you remember. For multi-agent and multi-human modes, losing the audit trail makes debugging and review harder.

### Alternatives

- **Manual session logs** -- Copy agent output into commit messages or a log file. Tedious but functional.
- **Screen recording** -- Captures everything but is not searchable or structured.
- **AI assistant built-in history** -- Most assistants save conversation history. Not git-native, not linked to specific commits, but better than nothing.

---

## Git Worktrees

### What it does for the methodology

Git worktrees let multiple agents work on the same repository simultaneously without conflicts. Each agent gets its own working directory with its own branch, sharing the same `.git` directory. The `skills/git-worktrees/` skill defines the conventions: naming, branch strategy, merge protocol.

This is the enabling mechanism for multi-agent mode (SCRUM.md). Each agent works in its own worktree. When an agent finishes, it merges its own branch back to main, runs the test suite, and pushes. Worktrees are about clean git history, not about needing supervision — agents are trusted to merge their own work.

### How the methodology degrades without it

Multi-agent mode still works with feature branches instead of worktrees. Each agent clones the repo separately or works on a branch in the same checkout (sequentially, not in parallel). The isolation is the same; the overhead is slightly higher.

Solo mode is unaffected. You don't need worktrees when there's one agent.

### Alternatives

- **Feature branches with separate clones** -- Same isolation, more disk space, separate `.git` directories.
- **Sequential work in one checkout** -- One agent at a time. No parallelism but no conflicts either.

---

## GitHub / GitLab

### What it does for the methodology

Code hosting and the PR workflow. Three things matter:

1. **Pull requests.** Multi-human mode (SCRUM.md) requires code review before merge. The `skills/code-review/` skill defines what a review checks. PRs provide the mechanism: diff view, inline comments, approval gates.

2. **CI/CD integration.** DEVOPS.md defines quality gates (tests pass, linter clean, build succeeds). GitHub Actions / GitLab CI enforces these gates automatically on every push and PR.

3. **Issue tracking.** Optional. The methodology uses PBIs (Product Backlog Items) managed however the team prefers -- a YAML file, a board, GitHub Issues, Jira. The toolchain doesn't prescribe issue tracking.

### How the methodology degrades without it

Without a PR workflow, code review becomes informal -- reviewing diffs locally, pair programming, or reviewing commits after merge. The code-review skill still defines what to check, but there's no structured place to do it.

Without CI integration, quality gates are manual. Someone has to remember to run tests before merging. The working agreements still apply, but enforcement is human discipline only.

Solo mode is barely affected. You're reviewing your own agent's work anyway. Multi-human mode takes the biggest hit because PR review is a core coordination mechanism.

### Alternatives

- **Gitea / Forgejo** -- Self-hosted, full PR workflow and CI (with Woodpecker or built-in Actions).
- **Bitbucket** -- PR workflow and Pipelines.
- **Any git hosting with PR support** -- The methodology needs diffs and review comments, not specific platform features.
- **No hosting platform** -- Bare git repo on a server. Works for solo. Awkward for teams.

---

## GitHub Actions / GitLab CI

### What it does for the methodology

CI automates the working agreements from DEVOPS.md:

- **Tests must pass before merge.** CI runs the test suite on every push. If tests fail, the branch can't merge. This enforces "no production code without a failing test first" -- if you skip TDD, CI catches missing coverage.
- **Linter must be clean.** Formatting and style checks run automatically. No debates about code style in review.
- **Warnings as errors.** Compiler and linter warnings treated as build failures. Prevents warning accumulation that hides real issues.
- **Build must succeed.** Catches compilation errors, missing dependencies, broken imports before they reach main.

The pipeline defined in DEVOPS.md maps directly to CI stages: lint, test, build, deploy. CI makes the pipeline real instead of aspirational.

### How the methodology degrades without it

Working agreements become trust-based. The developer (or agent) says "tests pass" and the reviewer believes them. The verification skill (`skills/verification/`) still requires evidence, but no system blocks a bad merge.

For disciplined solo developers, this is fine. For teams, it's a risk. Someone will eventually merge with failing tests, not out of malice but because they forgot to run them.

### Alternatives

- **Pre-commit hooks** -- Run tests and linters locally before commit. Faster feedback but bypassable (`--no-verify`).
- **Makefile targets** -- `make test`, `make lint`, `make build`. Manual but standardized.
- **Jenkins, CircleCI, Buildkite, Drone** -- Different CI systems, same concept. The methodology doesn't care which one runs the pipeline.

---

## Testing Frameworks

### What it does for the methodology

The TDD skill (`skills/tdd/`) is the backbone of the methodology's quality model. It requires a test runner -- some way to write a test, run it, see it fail, write code, see it pass. The specific framework matters less than having one.

The working agreement "no production code without a failing test first" is framework-agnostic. It works the same whether the test is written in Jest, pytest, ExUnit, Go's `testing` package, or plain shell scripts with assertions.

The TDD skill defines the process: red-green-refactor, test granularity, when to commit. The framework provides the execution environment.

### How the methodology degrades without it

The TDD skill cannot function. This is a hard dependency -- not on a specific framework, but on the *existence* of a test runner. Without tests, the working agreement is unenforceable and the verification skill has nothing to verify.

### Alternatives

Any test runner for your language. The methodology cares that tests exist and run, not how.

---

## ATDD/BDD Frameworks

### What it does for the methodology

Acceptance Test-Driven Development (ATDD) and Behavior-Driven Development (BDD) turn acceptance criteria into executable specifications. Feature files written in Gherkin (Given/When/Then) serve as both documentation and automated tests. The BDD skill (`skills/bdd/`) defines the process: write feature files before implementation, use them as the outer test loop while TDD drives the inner loop.

For AI-assisted development, BDD is especially valuable because:

1. **Business language in, code out.** A non-technical PO writes acceptance criteria in Gherkin. The agent translates them into step definitions and implements the code. The feature file is a contract both human and agent can read.
2. **Verification is built-in.** When all scenarios pass, the acceptance criteria are met — by definition. The verification skill has concrete evidence to point at.
3. **Living documentation.** Feature files stay up to date because they're executed on every build. Stale docs don't pass CI.

Common frameworks: Cucumber (Ruby, JS, Java), Behave (Python), Godog (Go), Wallaby/ExUnit (Elixir), SpecFlow (.NET).

### How the methodology degrades without it

Acceptance criteria remain prose that someone must verify manually. The gap between "what was asked" and "what was built" is bridged by human judgment instead of automated tests. The BDD skill still works as a conceptual approach (think in scenarios), but loses the automation that makes it powerful.

For non-technical solo users, this is the biggest loss — BDD frameworks are the primary way a non-technical PO can define and verify behavior without reading code.

### Alternatives

- **Integration tests written directly in the test framework** — Same coverage, less readable by non-developers.
- **Manual acceptance testing** — Works for small projects. Doesn't scale. Doesn't catch regressions.

---

## Linters and Formatters

### What it does for the methodology

Automated code quality. Linters catch bugs (unused variables, unreachable code, missing error handling). Formatters enforce consistent style (indentation, line length, import ordering).

In the CI pipeline (DEVOPS.md), linting is a gate: code that doesn't pass the linter doesn't merge. This eliminates an entire category of code review comments ("fix the formatting") and lets reviews focus on logic and design.

For AI agents specifically, linters catch a common failure mode: the agent generates syntactically valid but subtly wrong code (unused imports, shadowed variables, unchecked errors). The linter flags these before a human has to spot them.

**Agilar recommendation: warnings as errors.** Configure your linter and compiler to treat warnings as errors in CI. Agents tend to generate code that compiles but produces warnings. When warnings accumulate unchecked, they hide real issues. `--warnings-as-errors`, `-Werror`, `warnings_as_errors: true` — the flag name varies by language, the principle is universal.

### How the methodology degrades without it

Code review takes longer because reviewers catch style issues manually. Inconsistent formatting creeps in across files. Agent-generated code quality varies more because there's no automated check.

The degradation is real but not catastrophic. Teams survived for decades without linters.

### Alternatives

Any linter for your language. ESLint, Prettier, Black, gofmt, mix format, Clippy. The specific tool doesn't matter. Having *a* linter matters.

---

## Docker / Containers

### What it does for the methodology

Environment reproducibility. DEVOPS.md defines environments (development, staging, production) and requires parity between them. Docker makes parity achievable: the same container image runs in dev, staging, and production. "Works on my machine" becomes "works in the container."

For AI agents, containers provide a safe sandbox. An agent can run arbitrary commands inside a container without affecting the host system. This is especially relevant for the debugging skill (`skills/debugging/`) where the agent needs to reproduce issues in a controlled environment.

### How the methodology degrades without it

Environment drift becomes likely. Staging doesn't match production. A bug that appears in production can't be reproduced locally because the environments differ.

For small projects with simple dependencies (a single language runtime, a database), this might not matter. For anything with multiple services, system dependencies, or complex build steps, the lack of containers makes DEVOPS.md's environment parity aspirational rather than achievable.

### Alternatives

- **Nix / Devbox** -- Reproducible environments without containers. Different trade-offs (learning curve vs. isolation).
- **Vagrant** -- VM-based reproducibility. Heavier than containers but stronger isolation.
- **Manual environment management** -- Document dependencies and hope everyone follows the docs. Works until it doesn't.

---

## Infrastructure as Code

### What it does for the methodology

Optional layer on top of Docker/containers. Tools like Terraform, Pulumi, or Ansible define infrastructure as version-controlled files. Changes to infrastructure go through the same PR and CI workflow as code changes.

This extends DEVOPS.md's quality gates to infrastructure: infrastructure changes get reviewed, tested, and deployed through the pipeline, not applied manually by someone with SSH access.

### How the methodology degrades without it

Infrastructure changes happen outside the methodology's visibility. No review, no audit trail, no rollback mechanism beyond "remember what we changed and undo it manually."

For single-server deployments or platforms like Heroku/Railway that manage infrastructure for you, IaC is unnecessary overhead. For multi-server, multi-environment setups, it becomes increasingly valuable.

### Alternatives

- **Platform-as-a-Service** (Heroku, Railway, Fly.io) -- The platform manages infrastructure. No IaC needed.
- **Shell scripts** -- `deploy.sh` that SSHes into servers and runs commands. Not declarative, not idempotent, but simple and understandable.
- **Docker Compose** -- For single-host deployments, Compose files are effectively IaC. Simple, version-controlled, sufficient for many projects.

---

## Choosing Your Toolchain

Start minimal. Add tools when the pain of not having them exceeds the cost of adopting them.

**Bare minimum:** Git + a test runner + any AI assistant + the `skills/` directory as reference documentation. This gives you the methodology's principles with manual enforcement.

**Solo developer sweet spot:** Git + Claude Code + testing framework + BDD framework + linter (warnings as errors) + CI on push. Skills are executable, working agreements are automated, and one person can move fast with confidence.

**Team setup:** Everything above + PR workflow + Entire for audit trail. Multi-human mode needs the coordination mechanisms that PRs and CI provide.

**Multi-agent:** Everything above + git worktrees + sub-agent orchestration. This is the most automated configuration and benefits most from Claude Code's skills system and Entire's automatic capture.

The methodology doesn't change across these levels. The automation does.
