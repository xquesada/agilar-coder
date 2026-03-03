# Agilar DevOps: Pipeline, Environments & Quality Gates

How code moves from a developer's machine to production. Agilar's opinionated approach to branching, environments, quality gates, CI/CD, and observability — adapted per team mode.

## Branching Strategies

Branching strategy follows team mode. Pick the one that matches your configuration.

### Solo: Trunk-Based Development

All commits land on `main`. No branches, no PRs. The human is the only reviewer — they verify before committing.

```
main ──●──●──●──●──●──●──
```

Why it works: one human, one agent, no merge conflicts. The test gate and verification gate catch problems before they hit `main`. Adding branching overhead to a solo workflow is ceremony without value.

### Multi-Agent: Worktrees from Main

Each agent works in an isolated git worktree branched from `main`. When an agent finishes, it merges its own work back to `main`. Worktrees keep the git history clean — each agent's commits are on its own branch until merged.

```
main ──●──────────●──────●──
        \        / \    /
agent-1  ●──●──●    \  /
          \          \/
agent-2    ●──●──●──●
```

Worktrees provide filesystem isolation without the overhead of separate clones. Each agent sees a clean working directory. When done, the agent switches to `main`, merges its branch, runs the full test suite, and pushes. This is standard `git merge` — no special orchestration tooling required.

**Merge strategies** (all valid, pick what fits):

| Strategy | How it works | When to use |
|----------|-------------|-------------|
| **Agent self-merges** (default) | Agent merges its own branch to main when done | Independent PBIs, low conflict risk. Simplest. |
| **Lead session merges** | Session that spawned sub-agents merges after review | Using subagent-driven skill with review between tasks |
| **Human merges** | Human explicitly triggers merge | Maximum control over what reaches main |

#### Orchestrator Workflow

After a worker completes and review passes, the orchestrator follows this sequence:

```
Worker done → Review passes → Merge to main → Test suite on main → Push → CI check
                                                   |                        |
                                               Tests fail?              CI fails?
                                                   |                        |
                                              Fix before push         Spawn CI checker
```

1. **Merge** the worker's branch to main
2. **Run full test suite** on main — do not push if tests fail
3. **Push** to remote (if push permissions granted)
4. **Check CI** — if CI fails, dispatch the CI checker agent
5. **Report** — inform the human of the push result and CI status

**Push cost awareness:** Every push to remote may trigger CI pipelines (compute cost), send notifications (attention cost), and create permanent history (audit cost). Batch merges when practical — push once after multiple workers complete, not after each one.

See `skills/git-worktrees/` for the worktree management process and `skills/parallel-agents/` for patterns.

### Multi-Human: Feature Branches + PRs

Standard feature branch workflow. Each developer works on a branch, opens a PR, CI runs, a reviewer approves, branch merges to `main`.

```
main ──●──────────●──────────●──
        \        / \        /
feat-a   ●──●──●    \      /
          \          \    /
feat-b     ●──●──●──●──●
                PR → review → merge
```

PRs are mandatory. CI must pass before merge. Code review is not optional — see `skills/code-review/` for the review process.

| Aspect | Solo | Multi-Agent | Multi-Human |
|--------|------|-------------|-------------|
| Branch model | Trunk (`main`) | Worktrees from `main` | Feature branches |
| Merge mechanism | Direct commit | Agent self-merges (or lead session / human) | PR + review |
| Code review | Self (pre-commit) | Automated (TDD + verification) or human review | Peer review via PR |
| CI trigger | Pre-commit hooks | Post-merge on `main` | PR + post-merge |

## Source and Instance

Some software is self-contained — the code, configuration, and data all live together
in one project. A WordPress site with its wp-config.php is an example.

Other software separates **where you build it** from **where you run it**:

- The **source** is where developers work. It contains code, tests, and build
  instructions. It produces something you can run (a compiled program, a packaged
  app, a bundle).
- The **instance** is where the software actually runs. It contains configuration,
  secrets (passwords, API keys), and data created by users. It consumes what the
  source produced.

Think of it like a phone app: the developers write the code and publish it to the
App Store (source). You download it to your phone, log in with your account, and
your photos and messages live on your phone (instance). The developers never see
your data. Your phone never sees their source code.

### When to separate

| Situation | Recommendation |
|-----------|---------------|
| Simple website or prototype | All-in-one. Don't over-engineer. |
| App that runs on a server you manage | Consider separating. Config and secrets shouldn't be in the source repo. |
| Tool or service that runs on a different machine than where you develop | Separate. The source repo builds it, the target machine configures and runs it. |
| Software that multiple people or organizations will install | Always separate. Each installation is its own instance. |

### Rules when separated

1. The source repo never contains instance-specific configuration (server addresses,
   passwords, API keys, user data)
2. The instance never contains source code — only the built result and its configuration
3. The interface between them is explicit: an environment variable, a config file path,
   or a well-known directory structure

## Environments

Four environments, each with a specific purpose. Not every project needs all four from day one — the table below shows when each becomes mandatory.

| Environment | Purpose | When Mandatory |
|-------------|---------|----------------|
| **Dev** | Developer's machine or agent workspace. Write code, run tests, iterate. | Always |
| **Testing** | Merged code with test data and mocked external interfaces. PBI-by-PBI integration testing. | Multi-human mandatory. Solo and multi-agent recommended. |
| **Staging** | Production simulation. Sprint Review demos run here. | Any application already in production. |
| **Production** | Live system serving real users. No direct deploy without staging validation. | N/A |

### Dev

Every developer and every agent runs code locally. Tests execute here first. If it doesn't work in dev, it doesn't leave dev.

Requirements:
- Full test suite runs locally
- External services are mocked or stubbed
- Database can be reset to a known state
- Agent workspaces (worktrees, sandboxes) are disposable

### Testing

Merged code runs against test data with external interfaces mocked or sandboxed. This is where PBI-level integration testing happens — verifying that the new code works with everything else already merged.

For multi-human teams this is mandatory because multiple developers merge independently and integration issues surface here. For solo and multi-agent teams it's recommended but not required — the dev environment often covers enough when one person controls all changes.

### Staging

Production simulation. The staging workflow follows a strict pattern:

1. **Load** a fresh production backup into staging
2. **Deploy** the new build to staging
3. **Verify** the application works against real (copied) data
4. **Only then** deploy to production

This catches the class of bugs that only appear with real data shapes, real volumes, and real edge cases that test fixtures never cover.

Sprint Review demos run against staging, not production. Stakeholders see exactly what will ship, running against realistic data, without risk to the live system.

Future consideration: for enterprise contexts, add data scrubbing scripts between step 1 and step 2 to strip PII before loading production data into staging.

### Production

Live. Deployments arrive only after passing through staging. No exceptions, no "quick fix" direct deploys. The staging gate exists precisely for the moments when you're most tempted to skip it.

## Quality Gates

The working agreements from `README.md` are not philosophical principles — they are pipeline gates. Code that violates them does not advance.

### Gate 1: Test Gate

**All tests pass before commit.**

Every commit must leave the test suite green. In trunk-based (solo) development, this means running the full suite before every commit. In branch-based workflows, CI enforces this on every push.

This gate enforces the TDD working agreement at the pipeline level. See `skills/tdd/` for the test-driven development process.

| Team Mode | Enforcement |
|-----------|-------------|
| Solo | Pre-commit hook or manual discipline |
| Multi-agent | Agent runs full suite before merging its worktree to `main` |
| Multi-human | CI blocks PR merge on test failure |

### Gate 2: Review Gate

**Spec compliance and code quality review before merge.**

Code is reviewed for correctness, adherence to the plan, and quality before it reaches `main`. The reviewer depends on team mode.

See `skills/code-review/` for the review process.

| Team Mode | Reviewer |
|-----------|----------|
| Solo | Self-review (human verifies before commit) |
| Multi-agent | Automated review (TDD + verification) or human review |
| Multi-human | Peer review via PR |

### Gate 3: Verification Gate

**Fresh evidence before any completion claim.**

No PBI is marked done without running the code and observing the expected behavior. Screenshots, test output, or console logs — something concrete, generated after the final change. Stale evidence from a previous run does not count.

See `skills/verification/` for the verification process.

| Team Mode | Enforcement |
|-----------|-------------|
| Solo | Human confirms before marking PBI done |
| Multi-agent | Agent provides evidence before self-merging |
| Multi-human | PR includes verification evidence; reviewer confirms |

### Gate 4: Root Cause Gate

**No fix without understanding why.**

When a bug is found, the fix must address the root cause — not just suppress the symptom. If the developer (human or agent) cannot explain why the bug occurred, the fix is not ready.

See `skills/debugging/` for the four-phase debugging process.

### Gate 5: Staging Gate

**No production deploy without staging validation.**

Applies to any application already running in production. The staging workflow (load backup, deploy, verify) must complete successfully before the production deploy proceeds.

| Team Mode | Enforcement |
|-----------|-------------|
| Solo | Manual staging verification before production deploy |
| Multi-agent | Pipeline requires staging step before production deploy |
| Multi-human | CD pipeline enforces staging-before-production |

### Gate Summary

```
commit ──→ [Test Gate] ──→ [Review Gate] ──→ merge to main
                                                  │
                                    [Verification Gate]
                                                  │
                           build ──→ [Staging Gate] ──→ production
```

The root cause gate is not a pipeline step — it's a discipline gate that applies whenever a bug is being fixed, at any stage.

## CI Pipeline

Continuous Integration runs on every merge to `main` (and on every PR in multi-human mode). All checks are automated, all are required to pass.

### Checks

| Check | What It Does | Failure Means |
|-------|-------------|---------------|
| **Unit tests** | Run all unit tests | Broken logic. Fix before merge. |
| **Integration tests** | Run tests that cross module boundaries | Integration contract broken. |
| **Linting** | Enforce code style and conventions | Style violations. Auto-fix or manual fix. |
| **Type checking** | Static type analysis (where applicable) | Type errors. Fix before merge. |
| **Security scan** | Dependency vulnerability + static analysis | Known vulnerability. Evaluate severity, patch or document. |

### CI by Team Mode

| Team Mode | CI Trigger | Blocking? |
|-----------|-----------|-----------|
| Solo | Pre-commit hook (or post-push to `main`) | Yes — don't push broken code to `main` |
| Multi-agent | Pre-merge (agent runs suite before merging to `main`) | Yes — merge blocked until green |
| Multi-human | PR creation + update | Yes — PR cannot merge until CI passes |

## CD Pipeline

Continuous Deployment moves validated code through staging to production.

```
main (CI green) ──→ Build ──→ Deploy to Staging ──→ Verify ──→ Deploy to Production
```

### Steps

1. **Build** — Compile, bundle, or package the application. Produce a versioned, immutable artifact.
2. **Deploy to Staging** — Load fresh production backup into staging database. Deploy the new build.
3. **Verify on Staging** — Run smoke tests and manual verification against staging. Confirm the application works with real data.
4. **Deploy to Production** — Deploy the same artifact that was validated in staging. No rebuilding.

### CD by Team Mode

| Team Mode | Deploy Mechanism | Who Triggers Production Deploy |
|-----------|-----------------|-------------------------------|
| Solo | Script or manual | Human |
| Multi-agent | Script | Human approves; agent executes |
| Multi-human | Automated pipeline | Pipeline after staging verification passes |

The artifact deployed to production must be the same artifact validated in staging. Rebuilding between staging and production defeats the purpose of staging validation.

## CI Auto-Repair (Multi-Agent)

In multi-agent mode, CI failures can be diagnosed and fixed automatically by a dedicated CI checker agent. The orchestrator dispatches the CI checker after a push triggers a failing CI run.

### Workflow

```
Push to remote
       |
       v
   CI runs ──pass──> Done (report success)
       |
      fail
       |
       v
  Spawn CI checker agent
       |
       v
  Monitor: read CI failure output
       |
       v
  Diagnose: categorize failure (test/lint/build/security)
       |
       v
  Fix: apply minimal targeted fix
       |
       v
  Verify: run the failed check locally
       |
      pass ──> Commit fix, merge to main, push ──> CI runs again
       |                                                |
      fail                                          pass ──> Done
       |                                                |
  Retry (max 3) ──> Escalate to human               fail ──> Loop
```

### Safety Rules

1. **Fix the code, not the pipeline** — Never modify CI configuration files (`.github/workflows/`, `.gitlab-ci.yml`). If the pipeline definition is wrong, escalate to the human.

2. **Never skip hooks or verification** — The CI checker follows the same working agreements as every other agent. No `--no-verify`, no force push, no skipped tests.

3. **Never change test assertions** — If a test fails, the test is telling you something. Fix the code under test, not the assertion. If the assertion is genuinely wrong (the spec changed), escalate — that's a scope decision, not a CI fix.

4. **Minimal fixes only** — The CI checker's job is to make the failing check pass without changing anything else. No refactoring, no improvements, no "while I'm here" changes.

5. **3 retries maximum** — If the CI checker cannot fix the failure in 3 attempts, something is structurally wrong. Escalate to the orchestrator or human with a diagnosis report.

6. **Commit hygiene** — CI fix commits use a clear prefix: `ci-fix: [description]`. This makes them easy to identify in git history and during code review.

### Integration with Quality Gates

CI auto-repair operates between Gate 1 (Test Gate) and Gate 2 (Review Gate). The CI checker's fixes are still subject to review — they are not exempt from the working agreements. The orchestrator should review CI fix commits before pushing again, or dispatch the code reviewer to verify the fix is correct.

### When NOT to Auto-Repair

- **Flaky tests** — If a test fails intermittently, the fix is not a retry. Investigate the root cause (see `skills/debugging/`).
- **Infrastructure failures** — CI runner out of disk, network timeout, service unavailable. These are not code problems. Report and wait.
- **New test failures on main** — If main was green before your push and is now red, your changes caused it. The CI checker fixes your changes, not pre-existing failures.

## Observability

Running software needs monitoring. AI-assisted development adds a new dimension: the agents themselves need observability.

### Application Observability

| Concern | What to Implement |
|---------|-------------------|
| **Health checks** | Every service exposes a health endpoint. Monitoring hits it on a schedule. Down is detected in minutes, not hours. |
| **State-transition alerting** | Alert on deployment status changes (deploying, deployed, failed, rolled back). Alert on service state changes (healthy, degraded, down). |
| **Logging** | Structured logs with correlation IDs. Queryable. Retained long enough to debug last week's incident. |

### Agent Observability

| Concern | What to Implement |
|---------|-------------------|
| **Audit trails** | Every agent action is logged — what it did, what files it changed, what commands it ran. Reconstructable after the fact. |
| **Cost monitoring** | Track token usage and cost per agent, per task, per time window. Set budget caps. Alert on anomalies. |
| **Quality metrics** | Track test pass rate, verification evidence rate, and working agreement violations over time. Degradation is a signal. |

Agent observability is not optional. An unsupervised agent without audit trails is a liability. The cost of logging everything is negligible compared to the cost of not knowing what happened.
