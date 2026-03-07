# Design: Deployment Configuration & Source/Instance Separation

**Date:** 2026-03-01
**PBIs:** #160 (Scaffold wizard: deployment environment configuration), #161 (Document source vs. instance separation)
**Status:** Approved

## Problem

The scaffold wizard generates a CLAUDE.md with working agreements, tech stack, backlog config, and team mode — but zero deployment configuration. DEVOPS.md describes environments conceptually (Dev/Testing/Staging/Prod) but the wizard never operationalizes it into the generated CLAUDE.md.

This means agents working in scaffolded projects have no idea how to deploy. They cannot "deploy to prod" without the human manually writing deploy instructions into CLAUDE.md after the fact.

Additionally, there is no concept of **source vs. instance separation** — the pattern where you build software in one place and run it in another with its own configuration, secrets, and data. This is well-understood in systems engineering but invisible to AI agents and non-technical users unless explicitly documented.

## Discovery

Found while deploying chepibe-whatsapp (a Go daemon) to M3. The engine code lives in chepibe-core (source repo), but it runs on M3 reading config from chepibe (instance repo). The agent building chepibe-core had no awareness of this separation — it added things to the Makefile but didn't know where or how to deploy.

## Design

### New Wizard Steps

Three new steps added after Step 7 (Code Review).

#### Step 8: Project Type

```
What are you building?

  1) A website or web page
     Examples: a company website like wikipedia.com, a blog, a landing page

  2) An app that people use in their browser
     Examples: something like Google Sheets, a dashboard, an online store

  3) A tool that runs in the background or from the command line
     Examples: a chatbot, a scheduled report, a file organizer

  4) I'll describe it myself (for experienced developers)
```

If option 4: capture free text, store verbatim in CLAUDE.md. The agent parses it at session start and asks clarifying questions if needed.

The project type drives Steps 9 and 10.

#### Step 9: Architecture (for types 1-3 only)

```
Where will the settings and data for this software live?

  1) Together with the code
     Like a Word document — everything in one place.
     Most projects start here.

  2) In a separate location
     Like an app you install on your phone — the app comes from
     the App Store, but your data and settings live on your phone.

  3) Not sure yet — I'll figure it out later
```

If option 2: ask `Describe where (a folder path, a server name, anything that helps):` — free text.

#### Step 10: Deployment (for types 1-3 only)

```
Where will this software run when it's ready for real use?

  1) On my own computer
  2) On another computer I manage (a server, a Raspberry Pi, etc.)
  3) On a hosting service (like Wix, Squarespace, Vercel, or similar)
  4) I don't know yet
```

If option 2: ask `How do you connect to that computer? (a name, address, or anything you know):` — free text.

If option 3: ask `Which service?:` — free text.

The wizard intentionally asks minimal questions — just the target. The agent discovers process managers, config paths, deploy commands, etc. at deploy time.

### Generated CLAUDE.md Sections

Two new conditional sections added to the generated CLAUDE.md.

#### Architecture section

Only generated when Step 9 = option 2 ("separate location").

```markdown
## Architecture: Source and Instance

This repo is the **source** — it contains the code you build. It does not contain
runtime configuration or data.

The **instance** (where the software actually runs) lives at: {user's description}

| What | Where |
|------|-------|
| Source code and tests | This repo |
| Configuration and settings | {user's description} |
| Data created at runtime | {user's description} |

When building, never add instance-specific configuration (secrets, server addresses,
user data) to this repo. When deploying, copy the built software to the instance
and let it read its configuration from there.
```

When Step 9 = option 1 (all-in-one) or option 3 (not sure yet), this section is omitted.

#### Deploy section

Generated for all project types. Content varies by combination of project type and deploy target.

**Website or browser app + runs on my computer:**

```markdown
## Deploy

This runs locally on this computer. Start it with:

\`\`\`bash
{build_cmd}
\`\`\`
```

**Website or browser app + another computer I manage:**

```markdown
## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer |
| Production | {user's description} |

To deploy, connect to the production machine and pull the latest code, build, and restart.
```

**Website or browser app + hosting service:**

```markdown
## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer |
| Production | {service name} |

Follow the hosting service's deployment process to publish updates.
```

**Background tool + another computer:**

```markdown
## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer (build and test here) |
| Production | {user's description} |

Build the software here, then deploy the result to the production machine.
```

**"I don't know yet" or "not decided":**

```markdown
## Deploy

Not configured yet. When you're ready to deploy, describe where the software should
run and the agent will help you set it up.
```

**"I'll describe it myself" (type 4):**

```markdown
## Deploy

{user's free text, verbatim}

The agent should use this description to understand the deployment setup. Ask
clarifying questions at session start if anything is unclear.
```

### DEVOPS.md Update

New section inserted **before** the existing "Environments" section.

```markdown
## Source and Instance

Some software is self-contained — the code, configuration, and data all live together
in one project. A WordPress site with its wp-config.php is an example.

Other software separates **where you build it** from **where you run it**:

- The **source** is where developers work. It contains code, tests, and build
  instructions. It produces something you can run (a compiled program, a packaged
  app, a bundle).
- The **instance** is where the software actually runs. It contains configuration,
  secrets (passwords, API keys), and data created by users. It consumes what the
  source produced.

Think of it like a phone app: the developers write the code and publish it to the
App Store (source). You download it to your phone, log in with your account, and
your photos and messages live on your phone (instance). The developers never see
your data. Your phone never sees their source code.

### When to separate

| Situation | Recommendation |
|-----------|---------------|
| Simple website or prototype | All-in-one. Don't over-engineer. |
| App that runs on a server you manage | Consider separating. Config and secrets shouldn't be in the source repo. |
| Tool or service that runs on a different machine than where you develop | Separate. The source repo builds it, the target machine configures and runs it. |
| Software that multiple people or organizations will install | Always separate. Each installation is its own instance. |

### Rules when separated

1. The source repo never contains instance-specific configuration (server addresses,
   passwords, API keys, user data)
2. The instance never contains source code — only the built result and its configuration
3. The interface between them is explicit: an environment variable, a config file path,
   or a well-known directory structure
```

### Updated Examples

#### solo-go (chepibe-core pattern)

Replace the hand-written "Project-Specific Notes" section with wizard-generated sections:

```markdown
## Architecture: Source and Instance

This repo is the **source** — it contains the code you build. It does not contain
runtime configuration or data.

The **instance** (where the software actually runs) lives at: /Users/4vq/projects/chepibe on M3

| What | Where |
|------|-------|
| Source code and tests | This repo |
| Configuration and settings | /Users/4vq/projects/chepibe on M3 |
| Data created at runtime | /Users/4vq/projects/chepibe on M3 |

When building, never add instance-specific configuration (secrets, server addresses,
user data) to this repo. When deploying, copy the built software to the instance
and let it read its configuration from there.

## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer (build and test here) |
| Production | M3 (4vq@100.91.174.29) |

Build the software here, then deploy the result to the production machine.
```

#### team-node (acme-dashboard pattern)

Replace the hand-written "Environments" section with:

```markdown
## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer |
| Production | Acme internal servers (staging.acme-dashboard.internal → dashboard.acme.com) |

To deploy, connect to the production machine and pull the latest code, build, and restart.
```

No Architecture section — this is all-in-one (code and config together).

## Scope Summary

| Change | File(s) |
|--------|---------|
| 3 new wizard steps (8, 9, 10) | `scaffold` |
| Architecture section generation | `scaffold` (CLAUDE.md template) |
| Deploy section generation | `scaffold` (CLAUDE.md template) |
| Source and Instance concept | `DEVOPS.md` |
| solo-go example update | `examples/solo-go/CLAUDE.md` |
| team-node example update | `examples/team-node/CLAUDE.md` |

## Design Principles

- **Non-technical language throughout** — the target audience is POs and business users, not sysadmins
- **Minimal wizard questions** — ask where, not how. The agent discovers the details at deploy time.
- **Type 4 is the power-user escape hatch** — experienced developers write free text, the agent parses it
- **Deploy sections are intentionally thin** — just enough for the agent to know where to look
