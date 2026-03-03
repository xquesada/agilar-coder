---
name: dispatching-parallel-agents
description: Use when facing 2+ independent tasks that can be worked on without shared state
---

# Dispatching Parallel Agents — Claude Code Implementation

This is the Claude Code implementation of the parallel agents skill. The canonical process definition is in `skills/parallel-agents/SKILL.md`. Read it first — this file adds Claude Code-specific execution details, not a separate process.

## How to Dispatch Parallel Agents

Use the **Agent** tool to dispatch multiple agents. Each call to **Agent** spawns an independent sub-agent that works in isolation.

**Key rule:** Make all parallel **Agent** calls in the same response. If you make them in separate responses, they run sequentially — defeating the purpose.

```
# CORRECT — parallel dispatch (same response, multiple Agent calls)
Agent call 1: "Investigate the auth middleware..."
Agent call 2: "Investigate the database connection pooling..."

# WRONG — sequential dispatch (separate responses)
Response 1: Agent call 1
Response 2: Agent call 2  ← this waits for call 1 to finish
```

## Agent Prompt Template

Each **Agent** call must include a self-contained prompt. The sub-agent has access to the filesystem and tools but has no conversation context — it only knows what your prompt tells it.

```
You are investigating/implementing [objective] as an independent task.

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
- [boundaries]

REPORT:
When done, provide:
- [specific deliverables]
- [format for findings/changes]
```

## Common Patterns

### Parallel Investigation

Dispatch agents to investigate different aspects of a problem. Read-only — no code changes.

```
Agent 1: "Investigate why the API returns 500 on /users endpoint.
          Read server/routes/users.js, server/middleware/auth.js, and
          server/db/queries/users.js. Check error handling paths.
          Report: root cause hypothesis with file paths and line numbers."

Agent 2: "Investigate the database connection timeout errors in production logs.
          Read server/db/pool.js, server/config/database.js. Check connection
          pool settings and timeout configuration.
          Report: current settings, recommended changes, reasoning."
```

### Parallel Implementation

Dispatch agents to implement independent features. Each agent should work in files that no other agent touches.

```
Agent 1: "Implement the /api/health endpoint.
          Create server/routes/health.js. Follow the pattern in
          server/routes/status.js. Write tests in tests/routes/health.test.js.
          Do NOT modify any existing route files."

Agent 2: "Implement the request logging middleware.
          Create server/middleware/request-logger.js. Follow the pattern in
          server/middleware/auth.js. Write tests in tests/middleware/request-logger.test.js.
          Do NOT modify any existing middleware files."
```

### Parallel Research

Dispatch agents to research different technologies or approaches for a design decision.

```
Agent 1: "Research SQLite as a storage option for this project.
          Check: max concurrent connections, WAL mode performance,
          Go driver maturity, backup strategies.
          Report: pros, cons, suitability assessment with specifics."

Agent 2: "Research PostgreSQL as a storage option for this project.
          Check: operational overhead for single-server deploy,
          Go driver maturity, connection pooling, backup strategies.
          Report: pros, cons, suitability assessment with specifics."
```

## After All Agents Complete

1. **Read each agent's report** — the **Agent** tool returns each agent's findings.

2. **Check for conflicts:**
   - Did any agent modify files it should not have?
   - Do findings contradict each other?
   - For implementation: did any agents accidentally overlap?

3. **Run integration checks:**
   - Use **Bash** to run the full test suite
   - Use **Grep/Glob** to verify no unexpected file changes
   - Use **Read** to review changed files if needed

4. **Synthesize** — combine findings or confirm implementation integration, then report to the user.

## Using Worktrees for Parallel Implementation

For parallel implementation tasks (not read-only investigation), use git worktrees to give each agent filesystem isolation. Use the **EnterWorktree** tool or dispatch agents that create their own worktrees.

Workflow:
1. Create a worktree per agent (or have each agent use **EnterWorktree**)
2. Each agent works in its isolated worktree
3. After completion, merge each worktree's branch back to `main`
4. Run the full test suite on `main` after all merges

This prevents filesystem conflicts when multiple agents write code simultaneously.

## Shared Resource Management

### Port Table in Worker Prompts

When dispatching agents that run servers, include port assignments:

```
PORT ASSIGNMENTS (your agent ID: 1):
- Dev server: 3001
- Test runner: 3101
- Do NOT bind to port 3000 or any port outside your range
```

### Browser Lock (Bash)

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

Include this pattern in worker prompts that involve E2E tests. Workers that don't use browsers don't need the lock.

### Database Isolation

For parallel agents that run tests with database writes:

```bash
# Each agent creates its own test database
export TEST_DB="testdb_agent_${AGENT_ID}"
createdb "$TEST_DB" 2>/dev/null || true
DATABASE_URL="postgres://localhost/$TEST_DB" npm test
dropdb "$TEST_DB" 2>/dev/null || true
```

## What NOT to Do

- Do not dispatch agents sequentially when they are independent — make all **Agent** calls in one response.
- Do not dispatch parallel agents for tasks that share files — they will conflict. Use `skills/subagent-driven/` for sequential execution.
- Do not skip the integration test suite after merging parallel work. Individual success does not guarantee combined success.
- Do not give agents prompts that reference other agents or their work. Each agent is isolated — it does not know the others exist.
- Do not use parallel dispatch for tasks where order matters. If task B depends on task A's output, run them sequentially.
