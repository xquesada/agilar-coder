# Sprint Planning

Take a refined PBI and break it into executable tasks that any engineer — human or AI — can follow with zero prior codebase context.

## Overview

This is Sprint Planning Part 2 — the activity where the team takes a refined PBI and decomposes it into a concrete implementation plan. The human designs (brainstorming, architecture decisions, acceptance criteria). Sprint planning translates that design into a sequence of concrete, executable tasks. The agent then follows the plan mechanically (see `skills/executing-plans/`).

In classic Scrum, Sprint Planning Part 1 is about WHAT to build (the PO presents refined PBIs). Part 2 is about HOW to build it (the team breaks PBIs into tasks). This skill is Part 2. Part 1 happens during Product Backlog Refinement (see `SCRUM.md`).

Plans are written AFTER the PBI meets the Definition of Ready (see `SCRUM.md`). If the PBI does not have acceptance criteria or an approved design, refine it first — do not plan unrefined work.

## When to Write a Plan

- The PBI involves more than one file or one logical step
- The PBI requires a sequence of changes that must be coordinated
- You want to hand off execution to a separate session or agent

If the PBI is small enough to implement in a single obvious step, skip the plan and just build it.

## PBI Size Check

Before writing the plan, estimate the total execution time. This check catches oversized PBIs early — before you invest time in detailed task breakdown.

**Heuristic:** Count the expected tasks at 2-5 minutes each. If the plan would exceed ~5-7 tasks (10-15 minutes of agent execution), the PBI is too large for a single plan.

**What to do when a PBI is too large:**

1. Stop before writing the plan
2. Suggest splitting the PBI into 2+ smaller PBIs, each independently deliverable
3. Explain what the natural split points are (e.g., "the API layer and the UI layer are independent deliverables")
4. The PO decides whether to split — the agent suggests, the human decides
5. If the PO decides to proceed without splitting, write the plan as-is (their call)

This check happens BEFORE writing the plan, not after. A plan that is too large is not a sign of a thorough plan — it is a sign of a PBI that was not broken down enough during refinement.

**Why this matters:** Large plans exhaust the agent's context window during execution. Beyond ~8 tasks, earlier task details get compressed, degrading quality. Smaller PBIs produce better plans, better execution, and more frequent delivery.

## Plan Document Structure

Every plan starts with a header that gives the executor full context:

```
# Plan: [PBI title]

PBI: #[id]
Goal: [One sentence — what the codebase looks like when this plan is complete]
Architecture: [Key design decisions — data model, API shape, component structure]
Tech Stack: [Languages, frameworks, libraries, test frameworks relevant to this plan]
Acceptance Criteria: [Copy from PBI — these are the success conditions]
```

The header exists so the executor never needs to go elsewhere for context. Everything needed to understand the "why" and "what" is right here.

## Plan Placement

The implementation plan is written as a `## Plan` section inside the PBI file at `backlog/pbi-NNN-short-description.md`. The plan follows the same task structure described below (Task N with File(s), What to do, Code, Test, Pre-commit, Commit).

The PBI file header (title, description, acceptance criteria) replaces the standalone plan header — it is already in the file. Do not duplicate the Goal, Architecture, Tech Stack, and AC in a separate `## Plan` header. The `## Plan` section starts directly with task definitions.

Design docs remain in `docs/plans/` as reference material (brainstorming output, cross-PBI architecture). PBI files reference them in their Notes section.

## Task Structure

Each task is a self-contained unit of work. Target 2-5 minutes of execution time per task. If a task would take longer, split it.

### Task Format

```
## Task N: [Short description]

**File(s):** [Exact file path(s) to create or modify]

**What to do:**
[Step-by-step instructions. Precise enough that the executor does not need to make design decisions.]

**Code:**
[Exact code to write, or a clear enough specification that the executor can write it without ambiguity.]

**Test:**
[Package-level test command and expected output. Every task must have a verification step.]

**Pre-commit:**
[Full suite: the project's test + lint commands (e.g., `make test && make lint`). Must pass before committing.]

**Commit:** [Commit message for this task]
```

### Task Granularity Rules

1. **One concern per task.** A task creates a type definition OR writes a function OR adds a test — not all three at once.
2. **Exact file paths.** Never say "create a file in the appropriate directory." Say exactly where.
3. **Exact code or unambiguous spec.** If the executor needs to make a judgment call, the plan is too vague.
4. **Every task has a verification step.** A test to run, a command to execute, or an expected output to observe.
5. **Every task has a commit message.** Frequent commits. Small, atomic, green.
6. **Every commit runs the full suite.** The pre-commit step runs the project's full test + lint commands — not just the tests for the current package. This catches cross-package breakages before they land on `main`.

### TDD Integration

Every task that produces application code follows the red-green-refactor cycle from `skills/tdd/`:

1. **Write the failing test.** The task specifies the test first. The executor writes it and runs it. It fails. This is expected.
2. **Write the minimal code.** The task specifies the production code. The executor writes it and runs the test. It passes.
3. **Full suite before commit.** Run the project's full test and lint commands (e.g., `make test && make lint`), not just the package-level tests. Trunk-based development means every commit lands on `main` — a package-level green is not enough; cross-package breakages must be caught before commit.
4. **Commit.** Green full suite, small commit.

A plan task might look like:

```
## Task 3: Add input validation for email field

**File(s):** tests/test_user_validation.py, src/user_validation.py

**What to do:**
1. Write a test that calls validate_email("not-an-email") and asserts it returns False
2. Write a test that calls validate_email("user@example.com") and asserts it returns True
3. Run the tests — both should FAIL (function does not exist yet)
4. Implement validate_email in src/user_validation.py using a regex pattern
5. Run the tests — both should PASS

**Test:**
  Run: python -m pytest tests/test_user_validation.py -v
  Expected: 2 passed

**Pre-commit:**
  Run: make test && make lint
  Expected: all green

**Commit:** "add email validation with tests"
```

Adapt the test framework and language to your project. The pattern is universal: test first, code second, verify, commit.

## Task Dependencies

Order tasks so each one builds on the previous. The executor follows them sequentially. If two tasks are independent, consider noting that they CAN run in parallel (separate agents, worktrees) but default to sequential.

A good dependency chain:

```
Task 1: Create the type/interface/schema
Task 2: Write tests for the core function (they fail)
Task 3: Implement the core function (tests pass)
Task 4: Write tests for the API endpoint (they fail)
Task 5: Implement the API endpoint (tests pass)
Task 6: Wire up the UI component
Task 7: End-to-end verification
```

## Plan Completeness Checklist

Before handing off the plan, verify:

- [ ] Every PBI acceptance criterion is addressed by at least one task
- [ ] Every task has exact file paths
- [ ] Every task has a verification step
- [ ] Every task that produces application code includes the TDD cycle
- [ ] Tasks are ordered by dependency
- [ ] The plan header contains the Goal, Architecture, Tech Stack, and Acceptance Criteria
- [ ] A final task verifies ALL acceptance criteria end-to-end
- [ ] Plan written in PBI file (`backlog/pbi-NNN-description.md`)
- [ ] PBI file moved to `backlog/ready/` after approval

## Execution Handoff

The execution approach depends on team mode. The agent decides — the user should not need to choose between execution strategies.

### Solo Mode

The agent picks based on plan size:

| Plan size | Approach | What the agent says |
|-----------|----------|---------------------|
| **Small (up to ~8 tasks)** | Same session, sequential | "Plan ready. Want me to start?" |
| **Large (9+ tasks)** | Fresh session with executing-plans skill | "This plan has N tasks. I recommend starting a fresh session to execute it — that way I won't lose context midway. The plan is in `backlog/ready/pbi-NNN-description.md`. Start a fresh session and tell it to build the next ready PBI." |

**Why 8 tasks:** Each task involves reading files, writing code, running tests, and producing output. Beyond ~8 tasks, context pressure causes compaction (earlier task details are summarized and compressed), which degrades quality. A fresh session gives the executor a clean context window dedicated to execution.

The agent may use sub-agents internally for isolated subtasks (e.g., writing a test file in parallel with a config file), but that is an implementation detail — not something the user decides.

After the plan is written and approved, the PBI file is moved to `backlog/ready/`. This signals to any executor (same session, fresh session, or sub-agent) that the PBI is ready for implementation.

### Multi-Agent Mode

Independent task groups are assigned to separate agents working in isolated worktrees. Each agent gets its assigned tasks from the plan. The lead agent coordinates via the task list and merges results. The orchestrator picks from `backlog/ready/`. See `skills/parallel-agents/` and `skills/git-worktrees/`.

### Multi-Human Mode

Task groups are assigned to developers. Each developer works in a feature branch, executes their tasks following `skills/executing-plans/`, and opens a PR when complete.

### For all paths

The executor follows `skills/executing-plans/`.

## Principles

- **DRY** — Do not Repeat Yourself. If two tasks need the same utility, the first task creates it.
- **YAGNI** — You Aren't Gonna Need It. Plan only what the acceptance criteria require. No speculative features.
- **TDD** — Test-Driven Development. Tests first, always. See `skills/tdd/`.
- **Frequent commits** — Every green test state is a commit opportunity. Small, atomic, descriptive.

## Anti-Patterns

- **Vague tasks.** "Set up the database" is not a task. "Create migration file `migrations/001_create_users.sql` with columns id, email, created_at" is a task.
- **Giant tasks.** If a task involves more than one logical concern, split it.
- **Missing verification.** If you cannot describe how to verify a task, the task is not well-defined.
- **Planning without acceptance criteria.** Go back to the PBI. Refine it first.
- **Gold-plating.** The plan delivers exactly what the acceptance criteria require. Nothing more.
