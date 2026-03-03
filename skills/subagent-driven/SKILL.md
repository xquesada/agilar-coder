# Subagent-Driven Development: Orchestrated Execution with Built-In Review

Execute an implementation plan by dispatching independent tasks to worker agents, with mandatory two-stage review between each task. The orchestrator coordinates, workers implement, reviewers verify.

This skill is the execution engine for plans produced by `skills/sprint-planning/`. It bridges the gap between "we have a plan" and "the plan is done" — with quality gates at every step.

## Working Agreement

**No task is complete without passing both spec compliance review and code quality review.** A worker agent's self-assessment is not sufficient. Two independent reviews must pass before advancing to the next task.

## When to Use

- You have an approved implementation plan with discrete, ordered tasks
- Tasks are largely independent (each can be implemented and tested without waiting for others)
- You want same-session execution with built-in quality control
- The plan was produced by `skills/sprint-planning/` or has equivalent structure

## When NOT to Use

- You are still in the design phase (use `skills/brainstorming/`)
- You do not have a plan yet (use `skills/sprint-planning/`)
- Tasks require deep exploration or research (use `skills/parallel-agents/` for investigation)
- The work is a single, indivisible change (just implement it directly with TDD)

## Roles

### Orchestrator

The coordinating agent. Manages the task queue, dispatches workers, dispatches reviewers, and decides when to advance. The orchestrator does not implement — it coordinates.

Responsibilities:
- Read the implementation plan and break it into dispatchable tasks
- Dispatch one worker per task with a focused, self-contained prompt
- Dispatch spec compliance reviewer after each task
- Dispatch code quality reviewer after spec review passes
- Handle review failures by re-dispatching the worker with feedback
- Track progress and report to the human partner

### Worker (Implementer)

A fresh agent dispatched for a single task. Receives a focused prompt, implements the task following TDD, and reports back.

Worker requirements:
- **Follow TDD** — write a failing test first, make it pass, refactor (see `skills/tdd/`)
- **Self-review before reporting** — re-read every file changed, check for mistakes, verify tests pass
- **Do nothing more, nothing less** than the task specifies
- **Report what was done** — list files changed, tests written, any deviations from the plan

A worker does not decide scope. A worker does not refactor adjacent code. A worker implements exactly what was asked and reports back.

### Spec Compliance Reviewer

A fresh agent that verifies the worker's output against the task specification. This reviewer answers one question: **did the worker implement exactly what was specified?**

Review checklist:
- Every requirement in the task spec is implemented
- Nothing beyond the task spec was implemented (no scope creep)
- File changes are limited to what the task required
- Test coverage matches the acceptance criteria
- No unrelated modifications were made

Verdict: **pass** or **fail with specific issues**. If fail, the issues go back to the worker.

### Code Quality Reviewer

A fresh agent that reviews the code for quality, independent of spec compliance. This reviewer answers: **is the code clean, correct, and well-tested?**

Review checklist:
- Code follows project conventions and patterns
- Tests are meaningful (not just "passes" — actually tests behavior)
- No dead code, debug artifacts, or commented-out code
- Error handling is appropriate
- Architecture boundaries are respected
- TDD discipline was followed (tests written before production code)

Verdict: **pass** or **fail with specific issues**. If fail, the issues go back to the worker.

## Process

### Step 1: Load the Plan

Read the implementation plan. Verify:
- The plan has discrete, ordered tasks
- Each task has clear acceptance criteria
- The plan was approved by the human partner
- The associated PBI exists and is in `in_progress` status

If any of these are missing, stop and resolve before proceeding.

### Step 2: Questions Before Implementation

After loading the plan and before dispatching the first worker, check for unanswered questions:

1. **Ambiguity check** — For each task, ask: "Could a reasonable developer interpret this two different ways?" If yes, clarify with the human partner before dispatching.

2. **Dependency check** — Are there implicit dependencies between tasks that the plan doesn't mention? (e.g., Task 3 creates a type that Task 5 uses, but the plan doesn't list this dependency)

3. **Scope check** — Does any task feel too large for a single worker dispatch? If a task has more than 3 requirements, consider splitting it.

4. **Convention check** — Does the project have patterns the workers need to follow? If so, include them explicitly in the worker prompt. Workers have no context beyond what you provide.

If any questions arise, resolve them with the human partner before proceeding. Dispatching a worker with ambiguous instructions wastes both time and tokens.

### Step 3: Dispatch the First Worker

Create a focused prompt for the first task. The worker prompt must include:

1. **Task description** — what to implement
2. **Acceptance criteria** — how to know it is done
3. **Context** — relevant files, patterns, constraints
4. **Constraints** — what NOT to do, boundaries
5. **TDD requirement** — write failing test first, make it pass, refactor
6. **Self-review requirement** — re-read all changes before reporting back
7. **Output format** — list files changed, tests written, deviations

The worker receives this prompt and executes independently.

### Step 4: Spec Compliance Review

After the worker reports back, dispatch a spec compliance reviewer. The reviewer prompt must include:

1. **Original task spec** — the exact requirements given to the worker
2. **Worker's report** — what the worker says it did
3. **Instruction** — verify the implementation matches the spec, nothing more, nothing less
4. **Files to review** — the files the worker changed

The reviewer examines the actual code changes against the spec and returns a verdict.

### Step 5: Handle Spec Review Result

**If pass:** Proceed to code quality review (Step 6).

**If fail:** Re-dispatch the worker with:
- The original task spec
- The reviewer's specific issues
- Instruction to fix only the identified issues

Then re-run the spec compliance review. Repeat until pass, up to 3 cycles. If 3 cycles fail, escalate to the human partner.

### Step 6: Code Quality Review

Dispatch a code quality reviewer. The reviewer prompt must include:

1. **Files to review** — all files changed by the worker
2. **Project conventions** — coding standards, patterns in use
3. **Review criteria** — clean code, meaningful tests, no dead code, proper error handling, architecture compliance

The reviewer examines code quality and returns a verdict.

### Step 7: Handle Code Quality Result

**If pass:** Task is complete. Advance to the next task (back to Step 3).

**If fail:** Re-dispatch the worker with:
- The quality reviewer's specific issues
- Instruction to fix only the identified issues
- Reminder not to change anything unrelated

Then re-run the code quality review. Repeat until pass, up to 3 cycles. If 3 cycles fail, escalate to the human partner.

### Step 8: Repeat Until Plan Complete

Continue dispatching workers and reviews for each task in the plan. After the final task passes both reviews:

1. Run the full test suite to verify nothing is broken
2. Compile a summary of all changes for the human partner
3. Provide verification evidence (see `skills/verification/`)
4. The human partner does a final review before marking the PBI done

## Review Sequencing

**Spec compliance review ALWAYS runs before code quality review.** Never reverse this order.

Why: If the implementation does not match the spec, code quality review is wasted effort. Fix what was built before polishing how it was built.

```
Worker implements
       |
       v
Spec Review ──fail──> Worker fixes ──> Spec Review (retry)
       |
      pass
       |
       v
Quality Review ──fail──> Worker fixes ──> Quality Review (retry)
       |
      pass
       |
       v
Next task
```

### Why Order Matters: A Waste Example

Imagine you run quality review first on a worker's output:

1. Quality reviewer finds 3 issues: naming conventions (Minor), missing error handling (Important), unused import (Minor)
2. Worker fixes all 3 issues
3. NOW spec compliance review runs: "This endpoint returns JSON but the spec says XML"
4. Worker must rewrite the entire endpoint — including re-fixing all 3 quality issues

Total: 2 quality reviews + 2 spec reviews + 2 worker dispatches = 6 agent calls
Correct order: 1 spec review + 1 quality review + 0-1 fix cycles = 2-3 agent calls

Wrong order literally doubles the cost. Spec compliance first, always.

## Worker Prompt Structure

A good worker prompt is self-contained. The worker should not need to explore the codebase to understand its task — the orchestrator provides the context.

Template:

```
TASK: [task title]

CONTEXT:
- [relevant file paths and their purpose]
- [relevant patterns, conventions]
- [dependencies on previous tasks, if any]

REQUIREMENTS:
1. [specific requirement]
2. [specific requirement]
3. [specific requirement]

CONSTRAINTS:
- Do NOT modify [files outside scope]
- Do NOT add [features not in spec]
- Follow the existing [pattern/convention] for [aspect]

PROCESS:
1. Write a failing test for the first requirement
2. Implement until the test passes
3. Refactor if needed
4. Repeat for each requirement
5. Self-review: re-read every file you changed
6. Run the full test suite

REPORT BACK:
- Files created or modified
- Tests written and their status
- Any deviations from the requirements (with justification)
```

## Red Flags

- **Skipping spec review because "it's obvious"** — the working agreement exists precisely for "obvious" tasks. Never skip.
- **Running quality review before spec review** — wrong order. Fix what was built before polishing how.
- **Worker making decisions about scope** — workers implement, they do not decide. Scope decisions go back to the orchestrator and ultimately the human partner.
- **Reviewer being vague** — "code could be better" is not actionable. Reviewers must cite specific files, lines, and issues.
- **More than 3 review cycles** — if a worker cannot satisfy a reviewer in 3 attempts, something is wrong with the task spec, the review criteria, or both. Escalate to the human partner.
- **Orchestrator implementing directly** — the orchestrator coordinates. If you find yourself writing code as the orchestrator, you have left the process.

## Batch Execution from Ready Queue

When told "work on ready PBIs" or "build all ready plans", the orchestrator:

1. **List** all PBI files in `backlog/ready/` (sorted by filename = PBI number order)
2. **Move** each to `backlog/in_progress/` and sync status to external tool (if configured)
3. **Evaluate independence:** Can PBIs execute in parallel? Check for shared files or dependencies between PBIs. If PBI A modifies `src/auth.go` and PBI B also modifies `src/auth.go`, they are dependent.
4. **Dispatch workers:**
   - **Independent PBIs** — dispatch workers in parallel (one PBI per worker, each in its own worktree)
   - **Dependent PBIs** — execute sequentially (finish PBI A before starting PBI B)
5. **Each worker** follows the full subagent-driven process for its PBI (dispatch → spec review → quality review)
6. **After each PBI completes:** move to `backlog/done/` and sync status to external tool

### Safety

If PBIs share files or dependencies, execute them sequentially — never in parallel. The cost of a merge conflict is higher than the cost of sequential execution.

### Reporting

The orchestrator reports completion of each PBI before starting the next:

```
PBI #42 (email validation) — complete. 5 tasks, all reviews passed.
PBI #43 (user profile) — starting now.
```

## Connection to Other Skills

| Skill | Relationship |
|-------|-------------|
| `skills/sprint-planning/` | Produces the implementation plans this skill executes |
| `skills/tdd/` | Workers follow TDD — tests before code, always |
| `skills/code-review/` | Quality reviewer follows code review principles |
| `skills/verification/` | Final verification evidence before PBI completion |
| `skills/parallel-agents/` | For investigation tasks, not implementation. Different pattern, different purpose |

## Connection to SCRUM.md

This skill executes PBIs. The orchestrator pulls a PBI that meets Definition of Ready, executes its plan task by task, and delivers a PBI that meets Definition of Done. The two-stage review process is how the Review Gate (see `DEVOPS.md`) is enforced during subagent-driven execution.

## Connection to DEVOPS.md

The two-stage review maps to Gate 2 (Review Gate). The final test suite run maps to Gate 1 (Test Gate). The verification evidence maps to Gate 3 (Verification Gate). Subagent-driven development does not bypass quality gates — it embeds them into every task.
