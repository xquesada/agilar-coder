---
name: dispatching-parallel-agents
description: Use when facing 2+ independent tasks that can be worked on without shared state. In Codex, execute tasks sequentially or use multiple Codex sessions with git worktrees for true parallelism.
---

<!-- Adapted from Claude Code: parallel agent dispatch replaced with sequential execution or multi-session worktree strategy. Codex cannot dispatch sub-agents, so parallelism requires external orchestration. -->

# Dispatching Parallel Agents — Codex Implementation

> Canonical skill: `skills/parallel-agents/SKILL.md`
>
> This file adapts the parallel agents pattern for Codex. Codex cannot dispatch multiple sub-agents in a single session. Two strategies are available: sequential execution within one session, or true parallelism via multiple Codex sessions with git worktrees.

## Strategy 1: Sequential Execution (Single Session)

When tasks are independent but you are working in a single session, execute them one at a time. The benefit over naive implementation is the structured prompt and strict scope isolation.

### Process

1. Identify independent tasks — verify no shared files or state
2. For each task, use the task prompt template below as a self-briefing
3. Execute each task completely before starting the next
4. After all tasks complete, run integration checks

### Task Prompt Template

Use this as a self-briefing checklist before each task:

```
OBJECTIVE: [one sentence]

SCOPE:
- Relevant files: [absolute paths]
- Module: [which part of the system]
- Boundary: do NOT modify anything outside [scope boundary]

CONTEXT:
- [background needed to understand the task]
- [current system state in this area]
- [known constraints]

TASK:
1. [specific step]
2. [specific step]
3. [specific step]

CONSTRAINTS:
- Do NOT modify files outside [scope]
- Do NOT change [aspects outside task]

DELIVERABLES:
- [specific deliverables]
- [format for findings/changes]
```

## Strategy 2: True Parallelism (Multiple Codex Sessions)

For tasks that genuinely benefit from parallel execution, use multiple Codex sessions, each in its own git worktree.

### Setup

```bash
# Create worktrees for each parallel task
git worktree add .worktrees/task-1 -b task-1 main
git worktree add .worktrees/task-2 -b task-2 main
```

Then start separate Codex sessions, each pointed at its own worktree directory. Each session receives the task prompt template above as its initial instruction.

### After All Sessions Complete

```bash
# Merge each task branch
cd /path/to/main/repo
git merge task-1
git merge task-2

# Run full test suite after merge
npm test  # or equivalent
```

## Common Patterns

### Parallel Investigation

Tasks that only read code — no modifications. Safe for sequential execution.

```
Task 1: "Investigate why the API returns 500 on /users endpoint.
         Read server/routes/users.js, server/middleware/auth.js, and
         server/db/queries/users.js. Check error handling paths.
         Report: root cause hypothesis with file paths and line numbers."

Task 2: "Investigate the database connection timeout errors in production logs.
         Read server/db/pool.js, server/config/database.js. Check connection
         pool settings and timeout configuration.
         Report: current settings, recommended changes, reasoning."
```

### Parallel Implementation

Tasks that write to different files. Use worktrees for true parallelism, or sequential execution if tasks are small.

```
Task 1: "Implement the /api/health endpoint.
         Create server/routes/health.js. Follow the pattern in
         server/routes/status.js. Write tests in tests/routes/health.test.js.
         Do NOT modify any existing route files."

Task 2: "Implement the request logging middleware.
         Create server/middleware/request-logger.js. Follow the pattern in
         server/middleware/auth.js. Write tests in tests/middleware/request-logger.test.js.
         Do NOT modify any existing middleware files."
```

### Parallel Research

Tasks that research different options. Always safe for sequential execution.

```
Task 1: "Research SQLite as a storage option for this project.
         Check: max concurrent connections, WAL mode performance,
         Go driver maturity, backup strategies.
         Report: pros, cons, suitability assessment with specifics."

Task 2: "Research PostgreSQL as a storage option for this project.
         Check: operational overhead for single-server deploy,
         Go driver maturity, connection pooling, backup strategies.
         Report: pros, cons, suitability assessment with specifics."
```

## After All Tasks Complete

1. **Review results from each task** — re-read all changed files and task outputs.

2. **Check for conflicts:**
   - Did any task modify files it should not have?
   - Do findings contradict each other?
   - For implementation: did any tasks accidentally overlap?

3. **Run integration checks:**
   - Run the full test suite in terminal
   - Search for unexpected file changes
   - Review changed files if needed

4. **Synthesize** — combine findings or confirm implementation integration, then report to the user.

## Shared Resource Management

### Port Assignments

When tasks run servers, assign non-overlapping ports:

```
Task 1:
- Dev server: 3001
- Test runner: 3101

Task 2:
- Dev server: 3002
- Test runner: 3102

Do NOT bind to port 3000 or any port outside your range.
```

### Browser Lock

For E2E tests that need a browser, use a filesystem lock:

```bash
# Acquire lock before browser tests
LOCK="/tmp/agilar-browser-lock"
while [ -f "$LOCK" ] && [ $(($(date +%s) - $(stat -f %m "$LOCK"))) -lt 300 ]; do sleep 2; done
echo $$ > "$LOCK"

# Run browser tests
npm run test:e2e

# Release lock
rm -f "$LOCK"
```

### Database Isolation

For parallel sessions that run tests with database writes:

```bash
# Each session creates its own test database
export TEST_DB="testdb_session_${SESSION_ID}"
createdb "$TEST_DB" 2>/dev/null || true
DATABASE_URL="postgres://localhost/$TEST_DB" npm test
dropdb "$TEST_DB" 2>/dev/null || true
```

## What NOT to Do

- Do not attempt to simulate parallelism within a single session — execute sequentially and be explicit about it.
- Do not use sequential execution for tasks that share files — they will conflict. Use the `subagent-driven` skill for ordered execution with dependencies.
- Do not skip the integration test suite after completing all tasks. Individual success does not guarantee combined success.
- Do not reference other tasks' work during execution. Each task is isolated — treat it as if the others do not exist.
- Do not use sequential execution for tasks where order matters. If task B depends on task A's output, that is a dependency chain — use `subagent-driven` instead.
