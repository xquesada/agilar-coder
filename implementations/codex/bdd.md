---
name: bdd
description: Generate BDD feature files and step definitions. Use when adding features with acceptance criteria that benefit from Given/When/Then structure.
---

# Behavior-Driven Development (BDD) — Codex Implementation

> Canonical skill: `skills/bdd/SKILL.md`

## Process

Follow the canonical skill process. Codex specifics below.

### 1. Write Feature Files

Read the PBI's acceptance criteria. Create feature files in the project's feature directory (commonly `features/`, `test/features/`, or `spec/features/`).

Create `.feature` files with Gherkin syntax.

### 2. Run Features — Verify Undefined Steps

Run the feature suite in terminal:

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

Create or edit step definition files. Keep steps thin — delegate to application code.

### 4. TDD Inner Loop

Follow the `test-driven-development` skill instructions for implementing the underlying code. BDD scenarios define WHAT; TDD defines HOW.

### 5. Verify All Pass

Run the full feature suite in terminal. Read the output completely before claiming success (per `verification-before-completion` skill).

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
