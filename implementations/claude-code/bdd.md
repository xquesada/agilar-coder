---
name: bdd
description: Generate BDD feature files and step definitions. Use when adding features with acceptance criteria that benefit from Given/When/Then structure.
---

# Behavior-Driven Development (BDD) — Claude Code Implementation

> Canonical process: `skills/bdd/SKILL.md`

## When to Use

**Always:**
- PBIs with user-facing behavior
- Features requiring stakeholder validation
- Integration scenarios spanning multiple components
- Acceptance criteria that benefit from Given/When/Then structure

**Exceptions (discuss with your human partner):**
- Pure internal refactoring (no behavior change)
- Infrastructure changes with no user-facing impact
- Utility functions tested adequately by unit tests

## Process

### 1. Write Feature Files

Read the PBI's acceptance criteria. Translate each criterion into one or more scenarios.

Use the Write tool to create `.feature` files in the project's feature directory (commonly `features/`, `test/features/`, or `spec/features/`).

```gherkin
Feature: Order placement
  As a customer
  I want to place an order for available products
  So that I can receive them at my address

  Scenario: Successful order with available stock
    Given the product "Widget" has 10 units in stock
    And I have "Widget" in my cart with quantity 2
    When I place the order
    Then the order status should be "confirmed"
    And the stock for "Widget" should be 8

  Scenario: Order rejected when insufficient stock
    Given the product "Widget" has 1 unit in stock
    And I have "Widget" in my cart with quantity 5
    When I place the order
    Then I should see an error "Insufficient stock for Widget"
    And no order should be created
```

**Feature file rules:**
- One feature per file, named after the feature: `order_placement.feature`
- Scenarios describe behavior, not implementation
- Use business language — no technical details in feature files
- Given = precondition, When = action, Then = expected outcome
- Each scenario tests one behavior (And in Then = additional assertions, not new behaviors)

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

All new scenarios should fail with "step not defined" or equivalent. **If a scenario passes immediately**, you're testing existing behavior — remove it or revise.

### 3. Implement Step Definitions

Use the Edit or Write tool to create step definitions. Keep steps thin — delegate to application code, don't implement business logic in steps.

### 4. TDD Inner Loop

Use the `tdd` skill for implementing the underlying code. BDD scenarios are the outer loop (WHAT); TDD is the inner loop (HOW).

```
BDD Scenario (outer loop)
  └── Step: "When I place the order"
       └── TDD cycle (inner loop)
            ├── RED: Write unit test for place_order()
            ├── GREEN: Implement place_order()
            └── REFACTOR: Clean up
```

### 5. Verify All Pass

Run the full feature suite with the Bash tool. Read the output completely before claiming success (per `verification` skill).

### 6. Map to Acceptance Criteria

Before claiming the PBI is done, explicitly map each acceptance criterion to its passing scenario(s). Every criterion must have at least one passing scenario. Report this mapping to your human partner.

## Feature File Conventions

### Scenario Outlines for Data Variations

```gherkin
Scenario Outline: Validate email format
  When I submit the form with email "<email>"
  Then I should see "<result>"

  Examples:
    | email              | result  |
    | user@example.com   | success |
    | invalid            | error   |
    | @no-local-part.com | error   |
```

### Background for Shared Preconditions

```gherkin
Feature: Shopping cart
  Background:
    Given I am logged in as "customer@example.com"
    And the store is open

  Scenario: Add item to cart
    ...

  Scenario: Remove item from cart
    ...
```

### Tags for Organization

```gherkin
@smoke
Scenario: Login with valid credentials
  ...

@slow @integration
Scenario: Full checkout flow
  ...
```

Use tags to run subsets (smoke, integration), mark by epic/PBI, or flag slow tests.

## Integration

| Skill | Integration |
|-------|------------|
| **TDD** | BDD is the outer loop, TDD is the inner loop |
| **Sprint Planning** | Plans reference feature files as acceptance criteria verification |
| **Verification** | Passing BDD scenarios ARE the verification evidence |
| **Code Review** | Reviewers check that scenarios cover all acceptance criteria |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Technical language in features | Rewrite in business language. No CSS selectors, no API paths. |
| Testing implementation, not behavior | Focus on WHAT happens, not HOW. "Order is saved to database" → "Order status is confirmed" |
| One huge scenario | Split into multiple scenarios, each testing one behavior |
| Brittle step definitions | Parameterize steps, use semantic matchers |
| Skipping the failing step | Run features first, confirm new steps are undefined |
| Business logic in steps | Steps should delegate to application code |

## Red Flags

- Writing features after implementation (same problem as tests-after in TDD)
- Scenarios that describe UI mechanics ("click button", "fill field") instead of behavior
- Step definitions with more than 5 lines of logic (too much in the glue layer)
- Features without connection to PBI acceptance criteria
- "It's too simple for BDD" — if it has acceptance criteria, it can have a feature file

## Verification Checklist

Before marking a PBI as done:

- [ ] Every acceptance criterion has at least one scenario
- [ ] All scenarios pass
- [ ] Step definitions are reusable and thin
- [ ] Feature files use business language only
- [ ] No implementation details leaked into features
- [ ] Scenarios were written before implementation
