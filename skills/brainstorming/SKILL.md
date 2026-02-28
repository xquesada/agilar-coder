# Brainstorming: Turning Ideas Into Designs

Help turn ideas into fully formed designs and specs through natural collaborative dialogue.

The human partner drives the conversation and makes all decisions. The AI agent proposes approaches, asks questions, explores the codebase, and drafts designs. This is the first step in the development lifecycle — nothing gets built until a design is approved.

## Working Agreement

**No implementation without an approved design.** Every project gets a design phase. No exceptions. No shortcuts. The human partner must explicitly approve the design before any code is written.

## Anti-Pattern: "This Is Too Simple To Need A Design"

Every project goes through this. A config change, a one-liner, a "quick fix." Resist it. "Simple" projects are exactly where unexamined assumptions cause the most wasted work. The design phase can be short — a few questions and a paragraph — but it cannot be skipped.

The discipline is the point. If the design takes two minutes, you lost two minutes. If it catches a wrong assumption, you saved two hours.

## Process

### Phase 1: Understand Context

Before asking a single question, understand what already exists.

- Read the project structure, recent commits, and relevant docs
- Identify existing patterns, conventions, and constraints
- Check the backlog for related PBIs — is there prior thinking on this topic?
- Understand the current state of the codebase in the area being discussed

Do not ask the human partner things you can discover yourself. They should be answering design questions, not giving you a tour of the repo.

### Phase 2: Ask Clarifying Questions

One question at a time. Do not dump a list of ten questions — that is lazy and shifts the cognitive load to the human partner.

Guidelines:
- **One question at a time** — wait for the answer before asking the next
- **Multiple choice preferred** — "Do you want A, B, or C?" beats "What do you want?" Give the human partner something to react to, not a blank page
- **Purpose before mechanics** — understand what problem is being solved before discussing how to solve it
- **Constraints before features** — what are the boundaries? What does this NOT do?
- **Success criteria** — how will we know this works? What does "done" look like?

Keep going until you can articulate the design. This might take three questions or fifteen. Do not rush it.

### Phase 3: Propose Approaches

Present 2-3 concrete approaches with honest trade-offs and your recommendation. For each approach:

- What it does and how it works
- Pros and cons (be specific, not generic)
- What you would need to build
- What you would NOT build (YAGNI ruthlessly)

Always include your recommendation and explain why. The human partner makes the decision, but you should have an opinion.

### Phase 4: Present the Design

Once the approach is chosen, present the full design. Scale each section to its actual complexity — a simple feature gets a short design, a complex system gets a detailed one. Do not pad simple designs with ceremony.

A design includes whatever subset of these sections is relevant:

- **Goal** — one sentence on what this achieves
- **Approach** — what we are building and how
- **Components** — what pieces exist, how they interact
- **Data model** — if there is persistent state
- **API/Interface** — if there are boundaries between components
- **Edge cases** — what could go wrong, how we handle it
- **Out of scope** — what we are explicitly not doing
- **Acceptance criteria** — concrete, testable conditions for "done"

**Get explicit approval.** Do not proceed until the human partner says some variant of "yes, build this." Silence is not approval. "Looks good" is approval. "I think so" needs a follow-up question.

### Phase 5: Write the Design Doc

Save the approved design to a file:

```
docs/plans/YYYY-MM-DD-<topic>-design.md
```

This document is the contract. It is what the implementation phase builds against. It is what code review checks against. Commit it to version control.

### Phase 6: Transition to Implementation

The approved design feeds directly into implementation planning:

- The acceptance criteria from the design become the PBI's acceptance criteria
- The design doc is referenced in the PBI
- The PBI meets Definition of Ready: it has a clear goal, acceptance criteria, and an approved design
- Next step: create an implementation plan (see the writing-plans skill)

## Connection to Definition of Ready

A PBI is "ready" when the team can pick it up and build it without ambiguity. Brainstorming produces the artifacts that satisfy DoR:

| DoR Criterion | Brainstorming Output |
|---------------|---------------------|
| Clear description | Goal and approach from the design |
| Acceptance criteria | Explicit in Phase 4 |
| Dependencies identified | Discovered during Phase 1-2 |
| Design approved | Hard gate in Phase 4 |
| Estimable | Trade-offs and scope clarified in Phase 3 |

If the brainstorming skill did its job, the PBI is ready. If the PBI is not ready, the brainstorming was incomplete.

## Key Principles

- **One question at a time** — respect the human partner's attention
- **Multiple choice preferred** — give them something to react to
- **YAGNI ruthlessly** — cut everything that is not needed for the stated goal
- **Explore alternatives** — always present more than one approach
- **Incremental validation** — get small confirmations along the way, not one big approval at the end
- **Be flexible** — the process serves the outcome, not the other way around
- **Human partner decides** — you propose, they dispose

## Red Flags

Stop and recalibrate if you notice:

- You are asking questions you could answer by reading the codebase
- You are presenting a single approach as if it were the only option
- The human partner is giving one-word answers (your questions are too open-ended)
- The design is longer than the code will be (over-engineering the ceremony)
- You are eager to start coding (that is the anti-pattern talking)
