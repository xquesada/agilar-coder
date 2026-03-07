# Behavior-Driven Development (BDD)

## Overview

Write executable specifications in business language before writing code. Features describe WHAT the system does from the user's perspective. Steps translate features into automated tests.

**Core principle:** Acceptance criteria become executable tests. If the feature file passes, the PBI is done.

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

## The Process

### 1. Write the Feature File

Start from the PBI's acceptance criteria. Translate each criterion into one or more scenarios.

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
- Each scenario tests one behavior (and in Then = additional assertions, not new behaviors)

### 2. Run Features — Watch Them Fail

Run the feature suite. All new scenarios should fail with "step not defined" or equivalent.

**If a scenario passes immediately:** You're testing existing behavior. Remove it or revise.

### 3. Implement Step Definitions

Write the glue code that connects Gherkin steps to actual code.

```
# Pseudocode — adapt to your framework
step "the product {name} has {count} units in stock" do
  create_product(name: name, stock: count)
end

step "I have {name} in my cart with quantity {count}" do
  add_to_cart(product: name, quantity: count)
end

step "I place the order" do
  @result = place_order(cart: current_cart)
end

step "the order status should be {status}" do
  assert @result.status == status
end
```

**Step definition rules:**
- Steps are reusable across scenarios
- Keep steps thin — delegate to application code, don't implement business logic in steps
- Use parameterized steps for flexibility
- Group related steps in the same file

### 4. Implement the Feature (TDD Inside)

With step definitions pointing at not-yet-existing code, use TDD (see `skills/tdd/`) to implement the underlying functionality. The BDD scenarios are the outer loop; TDD is the inner loop.

```
BDD Scenario (outer loop)
  └── Step: "When I place the order"
       └── TDD cycle (inner loop)
            ├── RED: Write unit test for place_order()
            ├── GREEN: Implement place_order()
            └── REFACTOR: Clean up
```

### 5. Run Features — Watch Them Pass

Run the full feature suite. All scenarios should pass.

**If a scenario fails:** Fix the implementation, not the scenario (unless the scenario was wrong).

### 6. Verify Against Acceptance Criteria

Map each acceptance criterion back to its scenario(s). Every criterion must have at least one passing scenario. This is the verification evidence for the PBI's Definition of Done (see SCRUM.md).

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

Use tags to:
- Run subsets of scenarios (smoke tests, integration tests)
- Mark scenarios by epic or PBI
- Flag slow tests for selective execution

## Integration with Other Skills

| Skill | Integration |
|-------|------------|
| **TDD** | BDD is the outer loop, TDD is the inner loop. BDD scenarios drive what to build; TDD drives how to build it. |
| **Writing Plans** | Plans reference feature files as acceptance criteria verification |
| **Verification** | Passing BDD scenarios ARE the verification evidence for PBI acceptance criteria |
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
