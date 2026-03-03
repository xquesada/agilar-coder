# Entire Scaffold Step Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add Entire audit trail as step 8 in the scaffold wizard, generating a CLAUDE.md section + install hint when enabled.

**Architecture:** Four edits to one file (`scaffold`). Insert the prompt step, build the CLAUDE.md section variable, inject it into the heredoc, and update the summary output. No new files, no new dependencies.

**Tech Stack:** Bash (the scaffold is a single bash script)

**Design doc:** `docs/plans/2026-03-01-entire-scaffold-step-design.md`

---

### Task 1: Insert the Entire prompt step

**Files:**
- Modify: `scaffold:215-216`

**Step 1: Add step 8 between code review and project type**

After line 214 (`fi` closing code review) and before the current `# --- Step 8: Project type ---`, insert:

```bash
# --- Step 8: Audit trail ---

echo ""
echo -e "${BOLD}Audit trail:${NC}"
echo -e "  Entire captures a git-native log of every AI agent session (what was asked,"
echo -e "  what changed, what tests ran). Recommended for teams; optional for solo."
read -rp "Enable Entire? [y/N]: " ENTIRE_CHOICE
ENTIRE_CHOICE="${ENTIRE_CHOICE:-N}"
case "$ENTIRE_CHOICE" in
  [Yy]*) ENTIRE_ENABLED="yes" ;;
  *) ENTIRE_ENABLED="no" ;;
esac
```

**Step 2: Renumber subsequent step comments**

Change the three comments:
- `# --- Step 8: Project type ---` → `# --- Step 9: Project type ---`
- `# --- Step 9: Architecture (types 1-3 only) ---` → `# --- Step 10: Architecture (types 1-3 only) ---`
- `# --- Step 10: Deployment (types 1-3 only) ---` → `# --- Step 11: Deployment (types 1-3 only) ---`

**Step 3: Verify script syntax**

Run: `bash -n ~/projects/agilar-ai-sdlc/scaffold`
Expected: no output (syntax OK)

**Step 4: Commit**

```bash
cd ~/projects/agilar-ai-sdlc
git add scaffold
git commit -m "feat(scaffold): add Entire audit trail prompt as step 8"
```

---

### Task 2: Build the CLAUDE.md section variable

**Files:**
- Modify: `scaffold` (in the "Generate files" section, around line 415-422, after `CR_DOD` variable)

**Step 1: Add ENTIRE_SECTION variable builder**

After the `SIZE_UNIT` block (around line 429), insert:

```bash
# Build audit trail section
ENTIRE_SECTION=""
if [[ "$ENTIRE_ENABLED" == "yes" ]]; then
  ENTIRE_SECTION="## Audit Trail

**Entire** is enabled. Every AI agent session is captured as a git-native transcript:
what was asked, what the agent did, what files changed, what tests ran.

- Transcripts are stored in git history alongside the code changes
- The verification skill captures evidence automatically when Entire is active
- For retrospectives, use session transcripts to review what went well

Install: see [Entire documentation](https://github.com/entirejs/entire) for setup instructions."
fi
```

**Step 2: Inject into CLAUDE.md heredoc**

In the heredoc (the `cat > "$CLAUDEMD"` block), find:

```
$BRANCH_SECTION

$ARCH_SECTION
```

Change to:

```
$BRANCH_SECTION

$ENTIRE_SECTION

$ARCH_SECTION
```

**Step 3: Verify script syntax**

Run: `bash -n ~/projects/agilar-ai-sdlc/scaffold`
Expected: no output (syntax OK)

**Step 4: Commit**

```bash
cd ~/projects/agilar-ai-sdlc
git add scaffold
git commit -m "feat(scaffold): generate Audit Trail section in CLAUDE.md when Entire enabled"
```

---

### Task 3: Update the summary output

**Files:**
- Modify: `scaffold` (summary section, around line 599-619)

**Step 1: Add Entire to "What was created" block**

After the `docs/skills/` echo line (around line 599), insert:

```bash
if [[ "$ENTIRE_ENABLED" == "yes" ]]; then
  echo "  Entire                     — Git-native audit trail (install separately)"
fi
```

**Step 2: Update "Next steps" to include Entire install**

Change:

```bash
echo -e "${BOLD}Next steps:${NC}"
echo "  1. Review and customize CLAUDE.md for your project"
echo "  2. Set up your test runner and linter"
echo "  3. Create your first PBI"
echo "  4. Start building — the skills will guide the process"
```

To:

```bash
echo -e "${BOLD}Next steps:${NC}"
echo "  1. Review and customize CLAUDE.md for your project"
echo "  2. Set up your test runner and linter"
if [[ "$ENTIRE_ENABLED" == "yes" ]]; then
  echo "  3. Install Entire for automatic session capture (see CLAUDE.md)"
  echo "  4. Create your first PBI"
  echo "  5. Start building — the skills will guide the process"
else
  echo "  3. Create your first PBI"
  echo "  4. Start building — the skills will guide the process"
fi
```

**Step 3: Verify script syntax**

Run: `bash -n ~/projects/agilar-ai-sdlc/scaffold`
Expected: no output (syntax OK)

**Step 4: Commit**

```bash
cd ~/projects/agilar-ai-sdlc
git add scaffold
git commit -m "feat(scaffold): show Entire in summary and next steps when enabled"
```

---

### Task 4: Manual smoke test

**Step 1: Run wizard with Entire enabled**

Run: `~/projects/agilar-ai-sdlc/scaffold` in a temp directory, answer "y" to Entire. Verify:
- Step 8 prompt appears between code review and project type
- Generated CLAUDE.md contains `## Audit Trail` section after `## Branching`
- Summary shows Entire line
- Next steps include install instruction

**Step 2: Run wizard with Entire disabled**

Run again, answer "n" (or just press Enter) to Entire. Verify:
- No `## Audit Trail` section in CLAUDE.md
- No Entire line in summary
- Next steps has 4 items, not 5

**Step 3: Verify no blank lines leak**

When Entire is disabled, `$ENTIRE_SECTION` is empty. Check that CLAUDE.md doesn't have double blank lines between Branching and Architecture sections. If it does, wrap the heredoc injection:

```
$BRANCH_SECTION
$(if [[ -n "$ENTIRE_SECTION" ]]; then echo ""; echo "$ENTIRE_SECTION"; fi)

$ARCH_SECTION
```
