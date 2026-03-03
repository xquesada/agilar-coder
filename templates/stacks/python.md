# Python Stack

## Tech Stack

- **Language:** Python 3.10+
- **Package manager:** pip (or uv / poetry)
- **Virtual environment:** venv (or uv)

## Test Commands

```bash
# All tests
pytest

# Specific test file
pytest path/to/test_module.py

# Specific test
pytest path/to/test_module.py::test_my_function

# With coverage
pytest --cov=src --cov-report=term-missing

# Verbose
pytest -v
```

## Lint Commands

```bash
# Ruff (fast linter + formatter, replaces flake8 + isort + black)
ruff check .
ruff format .

# Type checking
mypy src/

# Or pyright for faster type checking
pyright
```

## Build Commands

```bash
# Package build
python -m build

# Or for applications, no build step — run directly
python -m myapp
```

## Recommended Configuration

### warnings-as-errors

```toml
# pyproject.toml
[tool.pytest.ini_options]
filterwarnings = ["error"]

[tool.mypy]
strict = true
warn_unused_configs = true
warn_return_any = true
warn_unreachable = true

[tool.ruff]
select = ["E", "F", "W", "I", "N", "UP", "B", "A", "SIM"]
```

### Test framework

pytest. Standard for Python. Supports fixtures, parametrize, and plugins.

### BDD framework

Behave (`behave`). Feature files in `features/`, step definitions in `features/steps/`.

Alternative: `pytest-bdd` for teams already using pytest.

### Pre-commit hook

```yaml
# .pre-commit-config.yaml (using pre-commit framework)
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.0
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: local
    hooks:
      - id: tests
        name: tests
        entry: pytest
        language: system
        pass_filenames: false
        always_run: true
```
