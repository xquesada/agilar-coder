# Facilitator: Running Scrum Events

Guide the team through Scrum events — sprint planning, daily standup, sprint review, sprint retrospective, and refinement sessions. Keep events focused, timeboxed, and outcome-oriented.

The facilitator manages the event structure, not the people. In solo and multi-agent modes, the facilitator guides a structured conversation between the human and agent(s). In multi-human mode, the facilitator supports a human Scrum Master in running the event.

## When to Use

| Event | Solo | Multi-Agent | Multi-Human |
|-------|------|-------------|-------------|
| **Refinement** | Yes — inline or scheduled | Yes | Yes |
| **Sprint Planning** | No — Kanban, no sprints | No — Kanban | Yes |
| **Daily Standup** | No — one person | No — one person | Yes |
| **Sprint Review** | No — continuous delivery | No — continuous delivery | Yes |
| **Retrospective** | Yes — periodic | Yes — periodic | Yes — every sprint |

Solo and multi-agent modes use only refinement and retrospectives. Multi-human mode uses all events.

## Event Facilitation

### Refinement

**Purpose:** Turn rough PBI ideas into ready items that meet the Definition of Ready.

**Duration:** Variable. Solo: 5-15 minutes per PBI. Multi-human: timeboxed session (1 hour max).

**Agenda:**
1. PO presents the PBI — what it is and why it matters
2. Clarifying questions — the team asks, the PO answers
3. Acceptance criteria — write them collaboratively (see `skills/po-coach/`)
4. Size check — can this be done in one session/sprint? If not, split it
5. Dependencies — does this depend on anything? Does anything depend on this?
6. Ready check — does it meet DoR? If yes, mark it `ready`. If not, identify what is missing.

**Facilitator's role:**
- Keep discussion focused on the current PBI — flag tangents and park them
- Ensure acceptance criteria are testable and specific
- Call the ready check — explicitly ask "does this meet DoR?"
- Capture decisions and action items

**Solo mode shortcut:** In solo mode, refinement can be a brief conversation: "Here is what I want to build. What questions do you have? Let's write the acceptance criteria." The structure is the same, just faster.

### Sprint Planning (Multi-Human Only)

**Purpose:** The team commits to a set of PBIs for the sprint and creates a Sprint Goal.

**Duration:** 1-2 hours for a 2-week sprint.

**Agenda:**
1. **Sprint Goal** — PO proposes a goal. What is the single most important outcome this sprint?
2. **Capacity** — How much can the team take on? Account for vacations, meetings, and known interruptions.
3. **Selection** — PO presents the top of the backlog. Team pulls PBIs they can commit to. Each PBI must meet DoR.
4. **Breakdown** — For each selected PBI, the team identifies tasks and discusses approach. Not a full design — just enough to know how to start.
5. **Commitment** — The team confirms the Sprint Backlog. This is a forecast, not a blood oath — but it should be realistic.

**Facilitator's role:**
- Enforce DoR — reject PBIs that are not ready
- Watch for over-commitment — teams consistently overestimate capacity
- Ensure the Sprint Goal is one sentence, not a paragraph
- Timebox: if planning takes too long, the backlog is not refined enough

### Daily Standup (Multi-Human Only)

**Purpose:** Synchronize the team. Identify blockers. Not a status report.

**Duration:** 15 minutes maximum. Hard stop.

**Format per developer:**
1. What did I do since last standup?
2. What will I do before next standup?
3. Any blockers?

**Facilitator's role:**
- Keep it to 15 minutes — ruthlessly cut long updates
- Park detailed discussions for after standup — "Let's take that offline"
- Flag blockers for immediate follow-up
- Agents can prepare standup summaries for their humans (what tests passed, what PBIs progressed, what issues were found)

**Anti-patterns to prevent:**
- Reporting to the Scrum Master instead of to each other
- Problem-solving during standup (park it)
- Going over 15 minutes
- Skipping blockers because "I'll figure it out"

### Sprint Review (Multi-Human Only)

**Purpose:** Demonstrate completed work to stakeholders. Get feedback. Adjust the backlog.

**Duration:** 1 hour for a 2-week sprint.

**Agenda:**
1. **Sprint Goal recap** — did we achieve it?
2. **Demo** — show completed PBIs against their acceptance criteria. Demo against staging, not production.
3. **Stakeholder feedback** — what works? What needs adjustment?
4. **Backlog impact** — capture feedback as new or updated PBIs. Re-prioritize if needed.

**Facilitator's role:**
- Ensure demos are against acceptance criteria, not just "look what we built"
- Capture feedback as concrete items — not vague impressions
- Keep it focused on completed work — do not demo in-progress items
- Manage time — allocate demo time proportional to PBI significance

### Sprint Retrospective

**Purpose:** Reflect on the process and identify one actionable improvement.

**Duration:** Solo: 15-30 minutes. Multi-human: 45-60 minutes.

**Format:**

**Step 1 — Gather data** (5-10 min)

Three columns:
- **What went well** — keep doing these things
- **What didn't go well** — stop or change these things
- **What to try** — experiments for next sprint/iteration

Each participant adds items silently, then shares.

**Step 2 — Discuss** (15-30 min)

For each column, discuss the items. Focus on patterns, not individual incidents. Look for:
- Working agreement violations — which agreements were hardest to follow? Why?
- Process friction — where did the methodology slow you down without adding value?
- Tool issues — what tooling gaps caused problems?
- Communication gaps — where did misunderstandings happen?

**Step 3 — Commit to action** (5-10 min)

Pick exactly ONE improvement to implement. Not three. Not five. One.

The improvement must be:
- **Specific** — "Write acceptance criteria before pulling PBIs" not "improve quality"
- **Measurable** — you can tell at the next retro whether it happened
- **Owned** — someone is responsible for it

**Facilitator's role:**
- Create psychological safety — blame-free discussion
- Keep discussion constructive — "what to change" not "who to blame"
- Force the single-improvement commitment — teams that pick five improvements achieve zero
- Follow up at next retro — did the improvement happen? What was the result?

**Solo mode retrospective:**

In solo mode, the retrospective is a structured self-reflection. The agent facilitates:

```
"Let's do a quick retro. Since the last one:

1. What went well? (What process steps saved you time or caught issues?)
2. What didn't go well? (Where did you lose time or skip process steps?)
3. What to try next? (One change to experiment with.)

What comes to mind?"
```

The agent captures the answers and the chosen improvement. At the next retro, the agent reminds the human of the previous improvement and asks whether it worked.

## Facilitation Principles

1. **Neutral.** The facilitator does not take sides, push specific decisions, or advocate for features. The facilitator manages the conversation, not the content.

2. **Timeboxed.** Every event has a time limit. Respect it. If the event is not done when time runs out, the preparation was insufficient — don't extend the event.

3. **Outcome-oriented.** Every event has a specific output. Refinement produces ready PBIs. Planning produces a Sprint Backlog. Review produces feedback. Retro produces one improvement. If the event ends without its output, it failed.

4. **Inclusive.** Everyone speaks. If someone is quiet, invite them. If someone dominates, redirect. In multi-human mode, ensure agents and humans both contribute.

5. **Parked items.** Discussions that go off-topic get parked — noted for follow-up, not lost. The facilitator maintains a parking lot and ensures parked items are addressed after the event.

## Connection to Other Skills

| Skill | Relationship |
|-------|-------------|
| `skills/po-coach/` | PO Coach activates during refinement (acceptance criteria) and review (acceptance). |
| `skills/scrum-master/` | Scrum Master enforces process between events. Facilitator manages events. |
| `skills/brainstorming/` | Brainstorming may happen during refinement for complex PBIs. |
| `skills/sprint-planning/` | Implementation plans may be drafted during sprint planning breakdown. |

## Anti-Patterns

**Event inflation:** Running all Scrum events for a solo developer. Solo mode needs refinement and retrospectives. That is all.

**Passive facilitation:** Letting discussions run without structure, timebox, or outcome. The facilitator's job is to manage the conversation actively.

**Retrospective theater:** Running retros without follow-through. If the improvement from the last retro was not tracked or discussed, the retro is ceremonial.

**Perfection over progress:** Spending too long refining a PBI to perfection. Good enough to start is good enough. The DoR is the bar, not perfection.
