# Deployment & Source/Instance Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add project type, architecture, and deployment steps to the scaffold wizard, update DEVOPS.md with the source/instance concept, and update both examples.

**Architecture:** Three new wizard steps (8, 9, 10) after the existing Step 7, conditional CLAUDE.md section generation, DEVOPS.md conceptual update, example updates.

**Tech Stack:** Bash (scaffold script), Markdown (DEVOPS.md, CLAUDE.md templates, examples)

---

### Task 1: Add Source and Instance section to DEVOPS.md

**Files:**
- Modify: `DEVOPS.md` — insert new section before line 65 (`## Environments`)

**Step 1: Insert the Source and Instance section**

Open `DEVOPS.md`. Find the line `## Environments` (line 65). Insert the following **before** it:

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

**Step 2: Verify the edit**

Read `DEVOPS.md` and confirm:
- The new "Source and Instance" section appears before "Environments"
- The existing "Environments" section and everything after it is unchanged
- No blank line issues or formatting problems

**Step 3: Commit**

```bash
git add DEVOPS.md
git commit -m "docs: add Source and Instance concept to DEVOPS.md (PBI #161)"
```

---

### Task 2: Add Step 8 (Project Type) to scaffold wizard

**Files:**
- Modify: `scaffold` — insert after the Step 7 code review block (after line 214)

**Step 1: Add the project type step**

Find the line `# --- Generate files ---` (line 216). Insert the following **before** it:

```bash
# --- Step 8: Project type ---

echo ""
echo -e "${BOLD}What are you building?${NC}"
echo "  1) A website or web page"
echo "     ${DIM}Examples: a company website like wikipedia.com, a blog, a landing page${NC}"
echo "  2) An app that people use in their browser"
echo "     ${DIM}Examples: something like Google Sheets, a dashboard, an online store${NC}"
echo "  3) A tool that runs in the background or from the command line"
echo "     ${DIM}Examples: a chatbot, a scheduled report, a file organizer${NC}"
echo "  4) I'll describe it myself (for experienced developers)"
read -rp "Choose [1]: " PROJECT_TYPE_CHOICE
PROJECT_TYPE_CHOICE="${PROJECT_TYPE_CHOICE:-1}"

PROJECT_TYPE_DESC=""
case "$PROJECT_TYPE_CHOICE" in
  1) PROJECT_TYPE="website" ;;
  2) PROJECT_TYPE="webapp" ;;
  3) PROJECT_TYPE="tool" ;;
  4)
    PROJECT_TYPE="custom"
    echo ""
    read -rp "Describe your software and how it should be deployed: " PROJECT_TYPE_DESC
    ;;
  *) echo "Invalid choice." >&2; exit 1 ;;
esac
```

**Step 2: Verify syntax**

```bash
bash -n scaffold
```

Expected: no output (clean parse).

**Step 3: Commit**

```bash
git add scaffold
git commit -m "feat(scaffold): add Step 8 — project type question"
```

---

### Task 3: Add Step 9 (Architecture) to scaffold wizard

**Files:**
- Modify: `scaffold` — insert after the Step 8 block just added, before `# --- Generate files ---`

**Step 1: Add the architecture step**

Insert after the Step 8 block, before `# --- Generate files ---`:

```bash
# --- Step 9: Architecture (types 1-3 only) ---

ARCH_TYPE="allinone"
INSTANCE_LOCATION=""
if [[ "$PROJECT_TYPE" != "custom" ]]; then
  echo ""
  echo -e "${BOLD}Where will the settings and data for this software live?${NC}"
  echo "  1) Together with the code"
  echo "     ${DIM}Like a Word document — everything in one place. Most projects start here.${NC}"
  echo "  2) In a separate location"
  echo "     ${DIM}Like an app you install on your phone — the app comes from the App Store,${NC}"
  echo "     ${DIM}but your data and settings live on your phone.${NC}"
  echo "  3) Not sure yet — I'll figure it out later"
  read -rp "Choose [1]: " ARCH_CHOICE
  ARCH_CHOICE="${ARCH_CHOICE:-1}"

  case "$ARCH_CHOICE" in
    1) ARCH_TYPE="allinone" ;;
    2)
      ARCH_TYPE="separated"
      read -rp "Describe where (a folder path, a server name, anything that helps): " INSTANCE_LOCATION
      ;;
    3) ARCH_TYPE="undecided" ;;
    *) echo "Invalid choice." >&2; exit 1 ;;
  esac
fi
```

**Step 2: Verify syntax**

```bash
bash -n scaffold
```

Expected: no output (clean parse).

**Step 3: Commit**

```bash
git add scaffold
git commit -m "feat(scaffold): add Step 9 — architecture question"
```

---

### Task 4: Add Step 10 (Deployment) to scaffold wizard

**Files:**
- Modify: `scaffold` — insert after the Step 9 block, before `# --- Generate files ---`

**Step 1: Add the deployment step**

Insert after the Step 9 block, before `# --- Generate files ---`:

```bash
# --- Step 10: Deployment (types 1-3 only) ---

DEPLOY_TARGET="local"
DEPLOY_TARGET_DESC=""
if [[ "$PROJECT_TYPE" != "custom" ]]; then
  echo ""
  echo -e "${BOLD}Where will this software run when it's ready for real use?${NC}"
  echo "  1) On my own computer"
  echo "  2) On another computer I manage (a server, a Raspberry Pi, etc.)"
  echo "  3) On a hosting service (like Wix, Squarespace, Vercel, or similar)"
  echo "  4) I don't know yet"
  read -rp "Choose [1]: " DEPLOY_CHOICE
  DEPLOY_CHOICE="${DEPLOY_CHOICE:-1}"

  case "$DEPLOY_CHOICE" in
    1) DEPLOY_TARGET="local" ;;
    2)
      DEPLOY_TARGET="remote"
      read -rp "How do you connect to that computer? (a name, address, or anything you know): " DEPLOY_TARGET_DESC
      ;;
    3)
      DEPLOY_TARGET="hosted"
      read -rp "Which service?: " DEPLOY_TARGET_DESC
      ;;
    4) DEPLOY_TARGET="undecided" ;;
    *) echo "Invalid choice." >&2; exit 1 ;;
  esac
fi
```

**Step 2: Verify syntax**

```bash
bash -n scaffold
```

Expected: no output (clean parse).

**Step 3: Commit**

```bash
git add scaffold
git commit -m "feat(scaffold): add Step 10 — deployment target question"
```

---

### Task 5: Generate Architecture section in CLAUDE.md

**Files:**
- Modify: `scaffold` — add architecture section generation inside the CLAUDE.md heredoc

**Step 1: Build the architecture section variable**

Find the line `# Build code review line` (around line 261, may have shifted after earlier inserts). Insert **before** it:

```bash
# Build architecture section (only for separated installs)
ARCH_SECTION=""
if [[ "$ARCH_TYPE" == "separated" ]]; then
  ARCH_SECTION="## Architecture: Source and Instance

This repo is the **source** — it contains the code you build. It does not contain
runtime configuration or data.

The **instance** (where the software actually runs) lives at: $INSTANCE_LOCATION

| What | Where |
|------|-------|
| Source code and tests | This repo |
| Configuration and settings | $INSTANCE_LOCATION |
| Data created at runtime | $INSTANCE_LOCATION |

When building, never add instance-specific configuration (secrets, server addresses,
user data) to this repo. When deploying, copy the built software to the instance
and let it read its configuration from there."
fi
```

**Step 2: Add the variable to the CLAUDE.md heredoc**

Find the line `$BRANCH_SECTION` inside the heredoc. Insert `$ARCH_SECTION` on a new line **after** the `## Branching` block and **before** `## PBI-First Rule`. The heredoc should look like:

```
## Branching

$BRANCH_SECTION

$ARCH_SECTION

## PBI-First Rule
```

Note: When `ARCH_SECTION` is empty, this produces a blank line which is harmless.

**Step 3: Verify syntax**

```bash
bash -n scaffold
```

**Step 4: Commit**

```bash
git add scaffold
git commit -m "feat(scaffold): generate Architecture section in CLAUDE.md"
```

---

### Task 6: Generate Deploy section in CLAUDE.md

**Files:**
- Modify: `scaffold` — add deploy section generation

**Step 1: Build the deploy section variable**

Find the architecture section variable code added in Task 5. Insert **after** it:

```bash
# Build deploy section
DEPLOY_SECTION=""
if [[ "$PROJECT_TYPE" == "custom" ]]; then
  DEPLOY_SECTION="## Deploy

$PROJECT_TYPE_DESC

The agent should use this description to understand the deployment setup. Ask
clarifying questions at session start if anything is unclear."
elif [[ "$DEPLOY_TARGET" == "undecided" ]]; then
  DEPLOY_SECTION="## Deploy

Not configured yet. When you're ready to deploy, describe where the software should
run and the agent will help you set it up."
elif [[ "$DEPLOY_TARGET" == "local" ]]; then
  DEPLOY_SECTION="## Deploy

This runs locally on this computer. Start it with:

\`\`\`bash
$BUILD_CMD
\`\`\`"
elif [[ "$DEPLOY_TARGET" == "remote" && "$PROJECT_TYPE" == "tool" ]]; then
  DEPLOY_SECTION="## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer (build and test here) |
| Production | $DEPLOY_TARGET_DESC |

Build the software here, then deploy the result to the production machine."
elif [[ "$DEPLOY_TARGET" == "remote" ]]; then
  DEPLOY_SECTION="## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer |
| Production | $DEPLOY_TARGET_DESC |

To deploy, connect to the production machine and pull the latest code, build, and restart."
elif [[ "$DEPLOY_TARGET" == "hosted" ]]; then
  DEPLOY_SECTION="## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer |
| Production | $DEPLOY_TARGET_DESC |

Follow the hosting service's deployment process to publish updates."
fi
```

**Step 2: Add the variable to the CLAUDE.md heredoc**

Find the `$BACKLOG_SECTION` line in the heredoc. Insert `$DEPLOY_SECTION` on a new line **after** it and **before** `## Skills`:

```
$BACKLOG_SECTION

$DEPLOY_SECTION

## Skills
```

**Step 3: Verify syntax**

```bash
bash -n scaffold
```

**Step 4: Commit**

```bash
git add scaffold
git commit -m "feat(scaffold): generate Deploy section in CLAUDE.md"
```

---

### Task 7: Update the wizard summary output

**Files:**
- Modify: `scaffold` — update the summary at the end of the script

**Step 1: Update the summary**

Find the `echo -e "${BOLD}What was created:${NC}"` block (near the end of the script). Add after the existing summary lines, before `echo -e "${BOLD}Next steps:${NC}"`:

```bash
if [[ "$ARCH_TYPE" == "separated" ]]; then
  echo "  Architecture: Source and Instance — instance at $INSTANCE_LOCATION"
fi
if [[ "$DEPLOY_TARGET" != "" ]]; then
  case "$DEPLOY_TARGET" in
    local)   echo "  Deploy: local (this computer)" ;;
    remote)  echo "  Deploy: $DEPLOY_TARGET_DESC" ;;
    hosted)  echo "  Deploy: $DEPLOY_TARGET_DESC" ;;
    undecided) echo "  Deploy: not configured yet" ;;
  esac
fi
if [[ "$PROJECT_TYPE" == "custom" ]]; then
  echo "  Deploy: custom (described in CLAUDE.md)"
fi
```

**Step 2: Verify syntax**

```bash
bash -n scaffold
```

**Step 3: Commit**

```bash
git add scaffold
git commit -m "feat(scaffold): show deploy config in wizard summary"
```

---

### Task 8: Update solo-go example

**Files:**
- Modify: `examples/solo-go/CLAUDE.md` — replace Project-Specific Notes with Architecture + Deploy sections

**Step 1: Replace the Project-Specific Notes section**

Find the `## Project-Specific Notes` section (line 97 to end of file). Replace it entirely with:

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

**Step 2: Verify**

Read the file and confirm the old "Project-Specific Notes" section is gone, replaced by the two new sections. Everything above (Working Agreements, Tech Stack, etc.) is unchanged.

**Step 3: Commit**

```bash
git add examples/solo-go/CLAUDE.md
git commit -m "docs: update solo-go example with Architecture and Deploy sections"
```

---

### Task 9: Update team-node example

**Files:**
- Modify: `examples/team-node/CLAUDE.md` — replace Environments section with Deploy section

**Step 1: Replace the Environments section**

Find the `## Environments` section (line 114 to end of file). Replace it entirely with:

```markdown
## Deploy

| Environment | Where |
|-------------|-------|
| Development | This computer |
| Production | Acme internal servers (staging.acme-dashboard.internal → dashboard.acme.com) |

To deploy, connect to the production machine and pull the latest code, build, and restart.
```

**Step 2: Verify**

Read the file and confirm the old "Environments" section is gone, replaced by the Deploy section. Everything above is unchanged. No Architecture section (this example is all-in-one).

**Step 3: Commit**

```bash
git add examples/team-node/CLAUDE.md
git commit -m "docs: update team-node example with Deploy section"
```

---

### Task 10: End-to-end verification

**Step 1: Run a dry test of the scaffold wizard**

Create a temp directory and run the wizard with different option combinations to verify the generated CLAUDE.md is correct:

```bash
mkdir -p /tmp/scaffold-test-1
```

Run the scaffold interactively, choosing:
- Project name: test-website
- Directory: /tmp/scaffold-test-1
- Team mode: 1 (Solo)
- Stack: 1 (Node)
- Tool: 1 (Claude Code)
- Backlog: 2 (YAML)
- Code review: Y
- Project type: 1 (website)
- Architecture: 1 (all-in-one)
- Deployment: 1 (local)

```bash
cd /path/to/agilar-coder && ./scaffold
```

Verify the generated CLAUDE.md:
- Has a Deploy section with local build command
- Does NOT have an Architecture section
- All other sections are present and correct

**Step 2: Test separated architecture + remote deploy**

```bash
mkdir -p /tmp/scaffold-test-2
```

Run the scaffold choosing:
- Project type: 3 (tool)
- Architecture: 2 (separate) → describe: "my-server at 192.168.1.100"
- Deployment: 2 (remote) → connection: "user@192.168.1.100"

Verify the generated CLAUDE.md:
- Has Architecture: Source and Instance section with "my-server at 192.168.1.100"
- Has Deploy section with "user@192.168.1.100" and "Build the software here" wording

**Step 3: Test custom type (type 4)**

```bash
mkdir -p /tmp/scaffold-test-3
```

Run the scaffold choosing:
- Project type: 4 (custom) → describe: "A Kubernetes operator that deploys to our EKS cluster via ArgoCD"

Verify the generated CLAUDE.md:
- Has Deploy section with verbatim text
- Does NOT have Architecture section
- Steps 9 and 10 were skipped

**Step 4: Clean up test directories**

```bash
rm -rf /tmp/scaffold-test-1 /tmp/scaffold-test-2 /tmp/scaffold-test-3
```

**Step 5: Final commit (if any fixups were needed)**

```bash
git add -A
git commit -m "fix(scaffold): fixups from end-to-end testing"
```

Only commit if changes were needed. If all tests passed clean, skip this step.
