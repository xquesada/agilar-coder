---
name: po-coach
description: "Use during backlog refinement, acceptance criteria writing, prioritization, or scope decisions. Helps the human partner be an effective Product Owner without replacing their judgment."
---

# PO Coach — Claude Code Implementation

> Canonical skill: `skills/po-coach/SKILL.md`
>
> This file adds Claude Code-specific workflow. The coaching techniques, anti-patterns, and principles in the canonical skill apply without modification. Read that file first.

## When This Activates

The PO Coach activates during product-level conversations:

- User says "let's figure out what to build next" → prioritization coaching
- User says "I need to write acceptance criteria" → criteria coaching
- User starts describing a large feature → help break it down
- User is creating or refining a PBI → refinement coaching
- User asks "is this PBI done?" → acceptance coaching
- User says "while we're at it..." mid-implementation → scope creep flag

## Backlog Access — Session Start Protocol

At session start, the PO Coach connects the agent to the backlog. This is the bridge between the PO layer (where requirements live) and the dev layer (where code is written).

### Step 1: Detect the backlog tool

Read the project's CLAUDE.md and find the `## Product Backlog` section. The section tells you which tool is used and how to access it. If no section exists, ask: "Where do you manage your Product Backlog?"

### Step 2: Execute the adapter

Based on what's configured in CLAUDE.md, follow the appropriate adapter below.

### Adapter: REST API (PO Companion / Chepibe)

The CLAUDE.md section contains the API base URL and filter criteria.

| Operation | Command |
|-----------|---------|
| **List ready PBIs** | `curl -s {base_url}` → parse JSON, filter items where `status == "ready"` and matching epic/project |
| **Read PBI details** | `curl -s {base_url}` → find item by `id` in response, read `notes`, `acceptance_criteria`, `checklist` |
| **Mark in_progress** | `curl -s -X PATCH {base_url}/{id} -H 'Content-Type: application/json' -d '{"status":"in_progress"}'` |
| **Mark done** | `curl -s -X PATCH {base_url}/{id} -H 'Content-Type: application/json' -d '{"status":"done"}'` |
| **Read acceptance criteria** | Same as Read PBI details — look for `acceptance_criteria` field or AC written in `notes` |
| **Update checklist** | `curl -s -X PATCH {base_url}/{id} -H 'Content-Type: application/json' -d '{"checklist":[...]}'` |

### Adapter: YAML File

The CLAUDE.md section contains the file path (e.g., `backlog.yaml`).

| Operation | How |
|-----------|-----|
| **List ready PBIs** | Read the file with the Read tool. Find items where `status: ready`. |
| **Read PBI details** | Find the item by `id` in the YAML. Read `notes`, `acceptance_criteria`, `checklist` fields. |
| **Mark in_progress** | Use the Edit tool to change `status: ready` to `status: in_progress` for the item. |
| **Mark done** | Use the Edit tool to change `status: in_progress` to `status: done` for the item. |
| **Read acceptance criteria** | Same as Read PBI details — look for `acceptance_criteria` field. |
| **Update checklist** | Use the Edit tool to update `done: false` to `done: true` for completed checklist items. |

### Adapter: GitHub Issues

The CLAUDE.md section identifies the repo (or it's the current repo).

| Operation | Command |
|-----------|---------|
| **List ready PBIs** | `gh issue list --label ready --json number,title,labels` |
| **Read PBI details** | `gh issue view {number}` — body contains acceptance criteria and checklist |
| **Mark in_progress** | `gh issue edit {number} --add-label in_progress --remove-label ready` |
| **Mark done** | `gh issue close {number}` |
| **Read acceptance criteria** | `gh issue view {number} --json body` — parse Given/When/Then blocks |
| **Update checklist** | `gh issue edit {number} --body "..."` — update task list checkboxes (`- [x]`) |

### Adapter: MCP-Based Tools (Jira, Miro, Linear)

The CLAUDE.md section identifies the project/board. The `.claude/settings.json` configures the MCP server connection. The MCP server exposes tool functions that map to the logical operations.

**Generic pattern:**

1. Read CLAUDE.md for project key / board ID / team identifier
2. Use the MCP tools available in your environment (they appear as regular tools)
3. Map the MCP tool functions to the 6 logical operations:
   - Search/list → List ready PBIs (filter by status and project)
   - Get/read → Read PBI details
   - Update/transition → Mark in_progress / Mark done
   - Get fields → Read acceptance criteria
   - Update fields → Update checklist

The exact tool names depend on the MCP server. For example, Jira MCP might expose `jira_search`, `jira_get_issue`, `jira_transition_issue`. Discover available tools and map them.

### What to Do at Session Start

1. Read `## Product Backlog` from CLAUDE.md
2. Use the appropriate adapter to list ready PBIs
3. Present the top ready PBI (or top 3 if multiple) with title and acceptance criteria
4. Ask: "Ready to start on this?" or let the user pick
5. When confirmed, mark it `in_progress`

## Acceptance Criteria Coaching

### Technique: Given/When/Then

When helping write acceptance criteria, propose them in Given/When/Then format:

```
Given [precondition or context]
When [the user does something or an event occurs]
Then [the expected outcome]
```

Use the user's own words. Do not rewrite their intent in technical jargon.

**Process:**
1. Read the PBI title and any existing notes
2. Ask: "What does success look like?"
3. Propose 2-3 acceptance criteria in Given/When/Then format
4. Ask: "Is anything missing? What about edge cases?"
5. Iterate until the user is satisfied
6. Update the PBI via the backlog API with the finalized criteria

### Edge Case Discovery

After the happy path is defined, systematically probe for edge cases:

- "What happens when the input is empty?"
- "What happens when the user is not logged in?"
- "What happens when the network is down?"
- "What happens when two users do this at the same time?"

Only ask relevant questions — do not run through a generic checklist. Read the PBI and ask about the edge cases that matter for this specific feature.

## Prioritization Coaching

When the user needs help deciding what to do next:

1. **Read the backlog** — fetch current PBIs via the API or read `backlog.yaml`
2. **Summarize the top candidates** — list the top 5-10 PBIs with their status and a one-line description
3. **Ask the value question** — "If you could only do 3 of these, which 3?"
4. **Challenge assumptions** — "Is PBI #X urgent, or does it just feel urgent?"
5. **Propose an order** — based on the conversation, suggest a priority order with reasoning
6. **Confirm** — let the user adjust, then update the backlog

## Breaking Down PBIs

When a PBI is too large:

1. Read the PBI's current description and acceptance criteria
2. Identify natural split points (see canonical skill for strategies)
3. Propose 2-4 smaller PBIs with clear boundaries
4. For each proposed PBI: title, 1-2 acceptance criteria, and the relationship to the original
5. Ask the user which split makes sense
6. Create the new PBIs via the backlog API and update or close the original

## Scope Creep Detection

During implementation, watch for scope expansion:

**Trigger phrases:**
- "While we're at it..."
- "Could we also..."
- "It would be nice to..."
- "One more thing..."
- New requirements appearing that were not in the acceptance criteria

**Response pattern:**
1. Acknowledge the idea — "Good idea."
2. Separate it — "That's separate from the current PBI."
3. Capture it — "Should I create a new PBI for it?"
4. Refocus — "Let's finish PBI #X first."

Do not argue about whether the new idea is valid. Just separate it from current work.

## Acceptance Walkthrough

When the user asks "is this done?" or when a PBI is being marked done:

1. Read the PBI's acceptance criteria from the backlog
2. For each criterion, present the evidence that it is met
3. Flag any criteria that are not clearly met
4. Ask: "Does this match what you asked for?"
5. If yes → suggest marking done. If no → identify what is missing.

This works with the verification skill — the PO Coach checks against requirements, the verification skill checks against test evidence.

## Claude Code-Specific Guidance

### Use the Backlog API

Always interact with PBIs through the project's backlog API when available. Do not suggest writing acceptance criteria "somewhere" — update the actual PBI.

### Read Before Asking

Before asking the user about their requirements, read:
- The PBI's current state (notes, checklist, acceptance criteria)
- Related PBIs in the same epic
- Any design docs referenced in the PBI
- Recent git commits related to the area

Do not ask the user to repeat information that is already in the system.

### Scale to Complexity

A one-line PBI ("Fix the typo on the login page") does not need acceptance criteria coaching. A multi-week feature does. Match the coaching effort to the PBI's complexity.

## What NOT to Do

- Do not write acceptance criteria without the user's input — propose, don't dictate
- Do not make scope or priority decisions — propose, ask, let the user decide
- Do not activate during implementation, testing, or code review — those belong to other skills
- Do not over-refine small PBIs — if it takes longer to write criteria than to do the work, the PBI is simple enough
- Do not generate a list of 20 probing questions — ask 2-3, iterate based on answers
