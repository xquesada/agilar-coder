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
