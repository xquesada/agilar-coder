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

## Orchestrator Responsibilities

When coordinating parallel agents, the orchestrator has specific duties beyond dispatch:

### Before Dispatch
1. **Create branches** — One branch per agent, created before dispatch. Branch naming: `agent-N/task-description` (e.g., `agent-1/add-health-endpoint`).
2. **Verify independence** — Confirm no file overlap between tasks (Step 1 of the Process).
3. **Prepare context** — Each agent prompt includes all files, patterns, and constraints. No assumptions about shared context.

### During Execution
4. **Monitor** — Check agent progress if tools allow. Do not interfere with running agents.
5. **Handle failures** — If an agent fails, assess whether to retry, reassign, or escalate.

### After Completion
6. **Review each result** — Read reports, check diffs, verify claims.
7. **Merge sequentially** — Merge one branch at a time to main. Run tests after EACH merge, not just after the last one.
8. **Push once** — After all merges pass tests, push to remote once. Batch pushes save CI runs.
9. **Cleanup** — Delete merged branches and worktrees.

## Shared Resources

Parallel agents may need access to shared resources: ports, databases, browsers. Uncoordinated access causes conflicts. The orchestrator assigns resources before dispatch.

### Port Allocation

Each agent gets a dedicated port range. The orchestrator assigns ports in the worker prompt.

| Agent | Dev Server Port | Test Port | Debug Port |
|-------|----------------|-----------|------------|
| Agent 0 (orchestrator) | 3000 | 3100 | 3200 |
| Agent 1 | 3001 | 3101 | 3201 |
| Agent 2 | 3002 | 3102 | 3202 |
| Agent 3 | 3003 | 3103 | 3203 |
| Agent 4 | 3004 | 3104 | 3204 |

Include port assignments in the worker prompt:
```
PORT ASSIGNMENTS:
- Dev server: 3001
- Test runner: 3101
- Do NOT use port 3000 (orchestrator) or any other agent's ports
```

### Database Safety

Parallel agents must NOT share a database instance for write operations. Options:
1. **Separate test databases** — Each agent gets its own test database (e.g., `testdb_agent1`, `testdb_agent2`)
2. **In-memory databases** — Each agent uses SQLite in-memory or equivalent
3. **Read-only shared access** — Multiple agents can read the same database safely; only one writes

### Browser Lock Pattern

If agents need a browser instance (for E2E tests), use a filesystem lock to prevent concurrent access:

```bash
LOCK_FILE="/tmp/browser-lock"
MAX_WAIT=60

acquire_lock() {
  local waited=0
  while [ -f "$LOCK_FILE" ]; do
    # Check for stale lock (older than 5 minutes)
    if [ -f "$LOCK_FILE" ] && [ $(($(date +%s) - $(stat -f %m "$LOCK_FILE" 2>/dev/null || echo 0))) -gt 300 ]; then
      rm -f "$LOCK_FILE"
      break
    fi
    sleep 2
    waited=$((waited + 2))
    if [ "$waited" -ge "$MAX_WAIT" ]; then
      echo "ERROR: Could not acquire browser lock after ${MAX_WAIT}s"
      exit 1
    fi
  done
  echo $$ > "$LOCK_FILE"
}

release_lock() {
  rm -f "$LOCK_FILE"
}

# Usage
acquire_lock
# ... run browser tests ...
release_lock
```

Agents include the lock in their E2E test scripts. The orchestrator does not manage the lock — agents self-coordinate through the filesystem.

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
