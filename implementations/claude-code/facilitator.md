---
name: facilitator
description: "Use when running Scrum events — refinement, sprint planning, standup, review, or retrospective. Manages event structure, timeboxes, and outcomes."
---

# Facilitator — Claude Code Implementation

> Canonical skill: `skills/facilitator/SKILL.md`
>
> This file adds Claude Code-specific execution details. The event formats, facilitation principles, and anti-patterns in the canonical skill apply without modification. Read that file first.

## How to Facilitate Events in Claude Code

Claude Code is a conversational agent, not a meeting tool. Facilitation happens through structured dialogue — the agent guides the conversation through the event's agenda and captures outcomes.

### Starting an Event

When the user requests or the Scrum Master nudges a Scrum event:

1. **Name the event** — "Let's run a [refinement / retro / planning session]."
2. **Set the scope** — "We'll focus on [specific PBIs / the last sprint / the upcoming sprint]."
3. **State the expected outcome** — "By the end, we should have [ready PBIs / a sprint backlog / one improvement to try]."
4. **Manage time** — "This should take about [X] minutes."

Then proceed through the agenda steps from the canonical skill.

### Refinement in Claude Code

**Solo/Multi-Agent mode — this is the most common event.**

1. Read the backlog — `curl` the backlog API or read `backlog.yaml` to identify PBIs that need refinement
2. Present the candidate — "PBI #X: [title]. Here's what we have so far: [existing notes/criteria]."
3. Ask clarifying questions — one at a time, multiple choice when possible (use the brainstorming skill's question style)
4. Write acceptance criteria — propose Given/When/Then format (delegate to PO Coach skill)
5. Check DoR — "Does this meet the Definition of Ready? Small enough, criteria written, design approved, no blockers?"
6. Update the PBI — use the backlog API to save criteria, update notes, and change status to `ready` if appropriate

### Sprint Planning in Claude Code (Multi-Human Only)

1. Read the backlog — fetch PBIs in `ready` status, sorted by priority
2. Present the Sprint Goal — ask the PO to propose one
3. Show capacity constraints — ask about team availability, known interruptions
4. Walk through ready PBIs — for each, ask "Include in this sprint?"
5. For selected PBIs, briefly discuss approach — "How would you tackle this?"
6. Summarize the Sprint Backlog — list the committed PBIs and the Sprint Goal
7. Capture the plan — save to a file or update PBI statuses

### Standup Summary (Multi-Human Only)

For each developer-agent pair, prepare a brief summary:

1. Use `git log --author=[name] --since=yesterday` to find recent commits
2. Check PBI statuses for in-progress items
3. Present a structured summary:

```
Since last standup:
- Completed: [PBI #X description]
- In progress: [PBI #Y description] — [status/blockers]
- Blockers: [any issues found]

Plan for today:
- Continue: [PBI #Y]
- Next: [PBI #Z]
```

### Sprint Review in Claude Code (Multi-Human Only)

1. Identify completed PBIs — fetch PBIs moved to `done` during the sprint
2. For each PBI, prepare a demo summary:
   - What was the acceptance criteria?
   - What was built? (link to relevant code/screenshots)
   - Does it meet the criteria?
3. Present to the user/stakeholders
4. Capture feedback as new PBIs or notes on existing PBIs

### Retrospective in Claude Code

**This works in all team modes.**

1. **Gather context** — use `git log` to summarize work since last retro
2. **Prompt reflection:**

```
Quick retro on the last [period]:

1. What went well? (Process steps that helped, things worth repeating)
2. What didn't go well? (Time lost, process skipped, frustrations)
3. What should we try differently?
```

3. **If a previous improvement was committed** — read it from memory/CLAUDE.md and ask: "Last time we committed to [improvement]. Did we follow through? What happened?"
4. **Help pick one improvement** — "Of everything we discussed, which one change would have the biggest impact?"
5. **Capture the improvement** — save it where the agent can find it next time (project memory, CLAUDE.md, or a dedicated retro log file)

## Parking Lot

During any event, when the conversation goes off-topic:

1. Acknowledge — "Good point."
2. Park — "Let me note that for after. [Save to a temporary list.]"
3. Redirect — "Back to [current agenda item]."
4. Follow up — at the end of the event, present parked items and ask what to do with each (create PBI, discuss now, discard).

In Claude Code, maintain the parking lot as a running list in your response. At the end of the event, present it.

## Capturing Outcomes

Every event must produce its defined output. Before ending:

| Event | Required Output | How to Capture |
|-------|----------------|----------------|
| Refinement | Ready PBIs | Update via backlog API |
| Sprint Planning | Sprint Backlog + Sprint Goal | Update PBI statuses + save goal |
| Standup | Blocker list | Flag blockers, create PBIs if needed |
| Sprint Review | Feedback as PBIs | Create/update PBIs via API |
| Retrospective | One improvement commitment | Save to memory or CLAUDE.md |

If the event ends without its output, say so: "We didn't reach a decision on [X]. Should we continue, or handle it separately?"

## What NOT to Do

- Do not run full Scrum events for solo/multi-agent mode — only refinement and retrospectives
- Do not facilitate in a formal, corporate style — match the user's communication style
- Do not generate fake standup reports — use real git and backlog data
- Do not end a retro without one concrete, owned improvement
- Do not facilitate implementation discussions during events — "Let's take the implementation details offline and discuss them during the brainstorming phase"
