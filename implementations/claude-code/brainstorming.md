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
- Read any existing PBI files in `backlog/` and ADRs in `docs/architecture/` that touch the same area

Do not ask the user things you can discover by reading files.

### Phase 2: Ask Clarifying Questions

Follow the canonical process: one question at a time, multiple choice preferred.

No special tooling needed. This is conversation.

### Phase 3: Propose Approaches

Follow the canonical process: 2-3 approaches with trade-offs and recommendation.

If an approach requires understanding feasibility, use the **Agent** tool to investigate specific technical questions before presenting (e.g., "check if library X supports feature Y" or "find how the existing auth middleware works").

### Phase 4: Present the Design

Follow the canonical process. Two sub-phases:

**4a: Design Concerns Scan.** Before writing the design, scan the concerns checklist from the canonical skill. Present the scan as a quick summary so the user can correct it:

> "Scanning design concerns: Architecture (relevant — new service), Data model (relevant — new table), Security (not relevant — internal tool), Performance (not relevant), UI (not relevant — backend only)..."

Use **Grep/Glob/Read** to inform the scan — check existing schemas, API patterns, and architecture before declaring something irrelevant.

**Version Impact.** Classify the PBI as major, minor, or patch. State it in the design: `**Version impact:** patch — bug fix, no API changes`. This informs the version bump at completion.

**4b: Elaborate the Design.** Write design sections for relevant concerns only. For the NFR checkpoint, state assumptions explicitly even when the answer is "not applicable."

**Hard gate.** Do not create any tasks, write any code, or transition to implementation until the user approves the design. "Looks good" counts. Silence does not.

### Phase 5: Write the Design Into the PBI File

The design goes INTO the PBI file, not into a separate document. Follow the sync workflow:

1. **Check the backlog tool** — does a PBI already exist for this work?
2. **Create in the tool first** if no PBI exists. Get the PBI number.
3. **Create or update the PBI file** (`backlog/pbi-NNN-description.md`):
   - Add `## Design` section with the approved design from Phase 4b
   - Add `## Acceptance Criteria` section with testable criteria
4. **Reference the file from the tool** — update the PBI's notes field with the file path.

Use **Write** or **Edit** tools to update the PBI file. Commit it immediately.

**Exception:** Cross-cutting architecture decisions that affect multiple PBIs → write to `docs/architecture/ADR-*.md` and reference from each PBI.

### Phase 6: Transition to Implementation

Once the design is approved and written to the PBI file:

1. The PBI file now has: Description, Design, and Acceptance Criteria — it meets Definition of Ready
2. Next step: invoke the **sprint-planning** skill to add a `## Plan` section to the same PBI file
3. The PBI file and PBI number are the single anchor for all subsequent work

## What NOT to Do

- Do not use TaskCreate during brainstorming. Tasks are for implementation, not exploration.
- Do not start writing code "just to prototype" before design approval. That is the anti-pattern.
- Do not spawn sub-agents for brainstorming conversation. The human partner is talking to you, not to a delegated agent.
- Do not skip Phase 3 (proposing approaches) even if the user seems to already know what they want. Present alternatives — they might change their mind.
