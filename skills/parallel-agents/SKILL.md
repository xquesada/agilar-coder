# Parallel Agents: Independent Tasks, Simultaneous Execution

Dispatch multiple agents to work on independent problems simultaneously. Each agent gets a focused, self-contained task. Results are reviewed and integrated after all agents complete.

This skill is primarily for multi-agent team mode (see `SCRUM.md`), where an orchestrator coordinates multiple workers. It applies to investigation, implementation, and analysis tasks that do not share state.

## Working Agreement

**No parallel dispatch without verified independence.** If two tasks share state, modify the same files, or depend on each other's output, they are not independent. Run them sequentially. Parallel execution of dependent tasks creates merge conflicts, race conditions, and wasted effort.

## When to Use

- You have 2 or more problems that can be investigated or implemented simultaneously
- Each problem lives in a distinct domain (different files, different modules, different concerns)
- No task needs the output of another task to proceed
- You want faster throughput by working on multiple fronts at once

Examples:
- Investigate a frontend bug AND a backend performance issue (different codebases)
- Implement 3 independent API endpoints that share no data model
- Research 2 unrelated technologies for a design decision
- Fix a CSS layout issue AND update a database migration (no overlap)

## When NOT to Use

- **Related failures** — if two bugs might share a root cause, investigate together (see `skills/debugging/`). Parallel investigation of related bugs leads to duplicate work or conflicting fixes.
- **Shared state** — if tasks modify the same files, database tables, or configuration, they will conflict. Run them sequentially.
- **Sequential dependencies** — if task B needs task A's output, they are not parallel. Use `skills/subagent-driven/` instead.
- **Need full system context** — if understanding the whole system is necessary, splitting the investigation loses the forest for the trees.
- **Single, cohesive change** — if the tasks are really one change broken into pieces, do not parallelize. The overhead of merging outweighs the speed gain.

## Process

### Step 1: Identify Independent Domains

Before dispatching, verify independence. For each pair of tasks, check:

- Do they modify the same files? If yes: not independent.
- Does one need the other's output? If yes: not independent.
- Do they share mutable state (database rows, config values, global variables)? If yes: not independent.
- Would a failure in one affect the other? If yes: possibly not independent.

If any pair fails the independence check, either make them sequential or restructure the tasks to eliminate the dependency.

### Step 2: Create Focused Task Prompts

Each agent prompt must be self-contained. The agent receives no context beyond what you provide. A good prompt includes:

1. **Objective** — what to accomplish, in one sentence
2. **Scope** — which files, modules, or areas are relevant
3. **Context** — background information needed to understand the task
4. **Constraints** — what NOT to do, boundaries, files to avoid
5. **Expected output** — what to report back (findings, code changes, analysis)

Common mistakes in parallel agent prompts:

| Mistake | Why It Fails | Fix |
|---------|-------------|-----|
| Too broad ("fix the frontend") | Agent flounders without direction | Specify which component, which behavior |
| No context ("fix the bug in auth") | Agent wastes time re-discovering what you already know | Include error messages, relevant file paths, reproduction steps |
| No constraints ("improve performance") | Agent changes things outside scope | Specify which files to touch, which to leave alone |
| Vague output ("let me know what you find") | You get a wall of text with no actionable structure | Specify format: "List each finding with file path, line number, and proposed fix" |
| Implicit dependencies ("also check if X works with the other agent's changes") | Agent cannot see the other agent's work | Remove cross-references. Each prompt is standalone. |

### Step 3: Dispatch All Agents

Dispatch all agents simultaneously. Do not wait for one to complete before dispatching the next — that defeats the purpose of parallelism.

Each agent works in isolation. It does not know about the other agents. It does not coordinate with them. It completes its task and reports back.

### Step 4: Review and Integrate Results

After all agents complete, review the results:

1. **Read each agent's report** — understand what was found or changed
2. **Check for conflicts** — did any agent unexpectedly modify files claimed by another? Did findings contradict each other?
3. **Resolve conflicts** — if two agents touched overlapping areas (despite independence checks), merge manually
4. **Run the full test suite** — verify that all changes work together (see `DEVOPS.md`, Gate 1: Test Gate)
5. **Synthesize findings** — if agents investigated different aspects of a problem, combine their findings into a coherent picture

### Step 5: Verify Integration

After merging parallel work, run verification to confirm nothing broke:

- Full test suite passes
- No regressions in areas adjacent to the changes
- Combined changes make sense as a whole (not just individually)

This is the Test Gate and Verification Gate from `DEVOPS.md` applied to parallel work integration.

## Parallel Agents and Git Worktrees

In multi-agent team mode, each parallel agent should work in its own git worktree (see `skills/git-worktrees/`). This provides filesystem isolation — each agent has its own working directory and branch. The orchestrator merges completed worktrees back to `main`.

```
main ──●──────────────●──
        \            /|
agent-1  ●──●──●──●   |
          \           /
agent-2    ●──●──●──●
```

Without worktrees, parallel agents would conflict on the filesystem. Worktrees are the mechanism that makes parallel implementation safe.

For parallel investigation (read-only tasks), worktrees are optional — agents can read from the same working directory without conflict.

## Task Prompt Template

```
OBJECTIVE: [one sentence — what to accomplish]

SCOPE:
- Files: [list relevant files or directories]
- Module: [which part of the system]
- Boundary: do NOT modify anything outside [scope boundary]

CONTEXT:
- [relevant background]
- [current state of the system in this area]
- [any known issues or constraints]

TASK:
1. [specific step]
2. [specific step]
3. [specific step]

CONSTRAINTS:
- Do NOT modify [files outside scope]
- Do NOT change [aspects outside task]
- [any other boundaries]

REPORT FORMAT:
- [what to include in the report]
- [expected structure]
- [specific data points needed]
```

## Red Flags

- **Dispatching with assumed independence** — if you did not verify independence (Step 1), you are gambling. Check first.
- **One agent waiting on another** — this means the tasks are not independent. Restructure or make them sequential.
- **Merge conflicts after parallel execution** — the independence check failed or was skipped. Fix the process, not just the conflict.
- **Agent reports referencing other agents' work** — prompts were not self-contained. Each agent should be unaware of the others.
- **Skipping the integration test suite** — individual task success does not guarantee combined success. Always run the full suite after merging parallel work.

## Connection to Other Skills

| Skill | Relationship |
|-------|-------------|
| `skills/subagent-driven/` | For sequential task execution with review gates. Use when tasks depend on each other. |
| `skills/git-worktrees/` | Provides filesystem isolation for parallel implementation. Essential for multi-agent team mode. |
| `skills/verification/` | Verification evidence required after integrating parallel results |

## Connection to DEVOPS.md

Parallel agents map to the multi-agent branching strategy (worktrees from main). The orchestrator is the integration point — it merges worktrees, runs the full test suite (Test Gate), and verifies the combined result (Verification Gate) before committing to `main`.
