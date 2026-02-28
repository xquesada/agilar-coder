---
name: brainstorming
description: "You MUST use this skill before any creative work — new features, architecture changes, designs, or anything that requires decisions. Do NOT skip this even for 'simple' tasks. Load and follow the process before writing any code."
---

# Brainstorming — Claude Code Implementation

This is the Claude Code implementation of the brainstorming skill. The canonical process definition is in `skills/brainstorming/SKILL.md`. Read it first — this file adds Claude Code-specific execution details, not a separate process.

## How to Execute Each Phase

### Phase 1: Understand Context

Use these tools to explore before asking questions:

- **Read** project structure, `CLAUDE.md`, recent git log, relevant source files
- **Grep/Glob** to find related code, patterns, and conventions
- **Agent** tool to delegate codebase exploration if the search space is large (e.g., "find all files related to authentication and summarize the current approach")
- Check `backlog.yaml` or the backlog API for related PBIs and prior design work
- Read any existing design docs in `docs/plans/` that touch the same area

Do not ask the user things you can discover by reading files.

### Phase 2: Ask Clarifying Questions

Follow the canonical process: one question at a time, multiple choice preferred.

No special tooling needed. This is conversation.

### Phase 3: Propose Approaches

Follow the canonical process: 2-3 approaches with trade-offs and recommendation.

If an approach requires understanding feasibility, use the **Agent** tool to investigate specific technical questions before presenting (e.g., "check if library X supports feature Y" or "find how the existing auth middleware works").

### Phase 4: Present the Design

Follow the canonical process. Get explicit user approval before proceeding.

**Hard gate.** Do not create any tasks, write any code, or transition to implementation until the user approves the design. "Looks good" counts. Silence does not.

### Phase 5: Write the Design Doc

Save the approved design:

```
docs/plans/YYYY-MM-DD-<topic>-design.md
```

Commit the design doc immediately after writing it. This is the contract for implementation.

### Phase 6: Transition to Implementation

Once the design is approved and committed:

1. Create a PBI via the backlog API if one does not already exist. Include:
   - Title and description from the design goal
   - Acceptance criteria from Phase 4
   - Reference to the design doc in notes
   - Checklist items for discrete implementation steps

2. Use **TaskCreate** to build the implementation plan as a tracked task list:
   - Break the design into ordered implementation steps
   - Each step should be independently testable
   - Include the TDD cycle: write test, make it pass, refactor
   - Reference the writing-plans skill for detailed planning if the implementation is non-trivial

3. The design doc path and PBI number become the anchor for all subsequent work.

## What NOT to Do

- Do not use TaskCreate during brainstorming. Tasks are for implementation, not exploration.
- Do not start writing code "just to prototype" before design approval. That is the anti-pattern.
- Do not spawn sub-agents for brainstorming conversation. The human partner is talking to you, not to a delegated agent.
- Do not skip Phase 3 (proposing approaches) even if the user seems to already know what they want. Present alternatives — they might change their mind.
