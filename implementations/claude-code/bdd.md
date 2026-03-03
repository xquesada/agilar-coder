---
name: bdd
description: Generate BDD feature files and step definitions. Use when adding features with acceptance criteria that benefit from Given/When/Then structure.
---

# Behavior-Driven Development (BDD) — Claude Code Implementation

> Canonical process: `skills/bdd/SKILL.md`

## Process

Follow the canonical skill process. Claude Code specifics below.

### 1. Write Feature Files

Read the PBI's acceptance criteria. Create feature files in the project's feature directory (commonly `features/`, `test/features/`, or `spec/features/`).

Use the Write tool to create `.feature` files with Gherkin syntax.

### 2. Run Features — Verify Undefined Steps

Use the Bash tool to run the feature suite:

```bash
# Cucumber (Ruby/JS)
npx cucumber-js features/order_placement.feature

# Behave (Python)
behave features/order_placement.feature

# ExUnit + Wallaby (Elixir)
mix test test/features/order_placement_test.exs

# Godog (Go)
godog features/order_placement.feature
```

Confirm new steps show as "undefined" or "pending".

### 3. Implement Step Definitions

Write step definitions using the Edit or Write tool. Keep steps thin — delegate to application code.

### 4. TDD Inner Loop

Use the `test-driven-development` skill for implementing the underlying code. BDD scenarios define WHAT; TDD defines HOW.

### 5. Verify All Pass

Run the full feature suite with the Bash tool. Read the output completely before claiming success (per `verification-before-completion` skill).

### 6. Map to Acceptance Criteria

Before claiming the PBI is done, explicitly map each acceptance criterion to its passing scenario(s). Report this mapping to your human partner.

## Integration

- **Feature files** = outer acceptance loop
- **Unit tests** (TDD) = inner implementation loop
- **Verification** = evidence that acceptance criteria are met
- **Code review** = verify scenario coverage matches acceptance criteria

## Red Flags

- Writing features after code (same problem as tests-after)
- Skipping the "undefined steps" verification
- Not mapping scenarios back to acceptance criteria
- Technical language in feature files
