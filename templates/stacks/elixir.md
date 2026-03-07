# Elixir Stack

## Tech Stack

- **Language:** Elixir
- **Framework:** Phoenix (if web) or plain Mix project
- **Build:** Mix
- **Package manager:** Hex

## Test Commands

```bash
# All tests
mix test

# Specific test file
mix test test/my_module_test.exs

# Specific test (by line number)
mix test test/my_module_test.exs:42

# With coverage
mix test --cover
```

## Lint Commands

```bash
# Format (auto-fix)
mix format

# Format (check only)
mix format --check-formatted

# Credo (static analysis)
mix credo --strict

# Dialyzer (type checking)
mix dialyzer
```

## Build Commands

```bash
# Compile
mix compile

# Release build
MIX_ENV=prod mix release
```

## Recommended Configuration

### warnings-as-errors

```elixir
# mix.exs
defmodule MyApp.MixProject do
  use Mix.Project

  def project do
    [
      # ...
      elixirc_options: [warnings_as_errors: true]
    ]
  end
end
```

### Test framework

ExUnit (built-in). No additional framework needed.

### BDD framework

Wallaby for browser-based acceptance tests. For Gherkin-style BDD, use `cabbage` or `white_bread`.

Alternative: ExUnit's built-in `describe`/`test` blocks with descriptive names often suffice for behavior-driven test organization.

### Pre-commit hook

```bash
#!/bin/sh
# .git/hooks/pre-commit
set -e
mix format --check-formatted
mix compile --warnings-as-errors
mix credo --strict
mix test
```
