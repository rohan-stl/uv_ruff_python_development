# UV + Ruff — Python 3.15+ Development Cheat Sheet

A practical reference for modern Python development using **uv** for Python version, environment, and dependency management; **Ruff** for linting and formatting; and **pytest** for automated testing.

> **Compatibility note:** Python 3.15 support depends on the selected Python release, uv version, Ruff version, and third-party dependencies. Verify compatibility before using Python 3.15 in production. For prerelease Python versions, expect some packages to require newer releases or source builds.

## Table of Contents

- [1. Installation](#1-installation)
- [2. Create a Python 3.15+ Project](#2-create-a-python-315-project)
- [3. Essential Daily Commands](#3-essential-daily-commands)
- [4. Ruff Configuration](#4-ruff-configuration)
- [5. Standard Development Workflow](#5-standard-development-workflow)
- [6. CI/CD Validation](#6-cicd-validation)
- [7. Git Practices](#7-git-practices)
- [8. Python 3.15+ Compatibility Practices](#8-python-315-compatibility-practices)
- [9. Official Documentation](#9-official-documentation)

---

## 1. Installation

Install uv on Ubuntu/Linux:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Verify the installation:

```bash
uv --version
uv python list
```

Install Python 3.15 if the desired release is available through uv:

```bash
uv python install 3.15
```

Verify the installed interpreter:

```bash
uv python find 3.15
uv run --python 3.15 python --version
```

If Python 3.15 is not yet available for your platform, use a supported interpreter version until the required release becomes available.

## 2. Create a Python 3.15+ Project

Create a project:

```bash
uv init my-python-app
cd my-python-app
```

Pin Python 3.15 for local development:

```bash
uv python pin 3.15
```

Configure the project's supported Python range in `pyproject.toml`:

```toml
[project]
name = "my-python-app"
version = "0.1.0"
requires-python = ">=3.15"
dependencies = []

[dependency-groups]
dev = [
    "ruff",
    "pytest",
]
```

Then synchronize the environment:

```bash
uv sync
```

Alternatively, add development dependencies through uv:

```bash
uv add --dev ruff pytest
```

**Important:** `requires-python = ">=3.15"` means Python 3.15 or later. Use this only if the application and its dependencies genuinely support that entire range. If Python 3.15 is the development target but older versions are also supported, declare the actual supported range instead.

### Recommended Project Structure

```text
my-python-app/
├── .venv/
├── src/
│   └── my_python_app/
│       └── __init__.py
├── tests/
│   └── test_example.py
├── .gitignore
├── .python-version
├── pyproject.toml
├── uv.lock
└── README.md
```

Commit `pyproject.toml`, `.python-version`, and `uv.lock` to Git. Do not commit `.venv/`.

## 3. Essential Daily Commands

### 3.1 Python Version Management

```bash
# Install Python 3.15
uv python install 3.15

# Pin the project interpreter
uv python pin 3.15

# Display available Python versions
uv python list

# Find the installed interpreter
uv python find 3.15

# Verify the interpreter used by the project
uv run python --version
```

For projects targeting multiple Python versions, validate the supported versions separately in CI.

### 3.2 Dependency Management

```bash
# Add a runtime dependency
uv add fastapi

# Add development dependencies
uv add --dev ruff pytest

# Remove a dependency
uv remove fastapi

# Synchronize the environment
uv sync

# Inspect the dependency tree
uv tree

# Check lockfile consistency
uv lock --check

# Upgrade dependencies deliberately
uv lock --upgrade
```

Before adding a dependency, verify that the selected package version supports Python 3.15.

### 3.3 Run Python Code

```bash
# Run a Python script
uv run python main.py

# Run a Python module
uv run python -m my_python_app

# Run a FastAPI application
uv run uvicorn app.main:app --reload

# Execute an inline command
uv run python -c "print('Hello, Python 3.15+')"
```

### 3.4 Ruff Linting

```bash
# Check the project
uv run ruff check .

# Fix supported linting issues
uv run ruff check --fix .

# Display rule documentation
uv run ruff rule F401
```

### 3.5 Ruff Formatting

```bash
# Format Python files
uv run ruff format .

# Check formatting without modifying files
uv run ruff format --check .

# Preview formatting changes
uv run ruff format --diff .
```

### 3.6 Testing with pytest

```bash
# Run all tests
uv run pytest

# Run a specific test file
uv run pytest tests/test_main.py

# Run one test
uv run pytest tests/test_main.py::test_example

# Verbose test output
uv run pytest -v
```

Check pytest and test-plugin compatibility with Python 3.15, especially when using a prerelease interpreter.

## 4. Ruff Configuration

Add the following configuration to `pyproject.toml`. Merge it with existing sections rather than creating duplicate TOML tables.

```toml
[tool.ruff]
line-length = 100
target-version = "py315"
src = ["src"]

[tool.ruff.lint]
select = [
    "E4",   # Selected pycodestyle errors
    "E7",
    "E9",
    "F",    # Pyflakes: common errors and unused imports
    "I",    # Import sorting
    "B",    # Bug-prone patterns
    "UP",   # Python syntax modernization
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["src"]
```

**Compatibility requirement:** Use `target-version = "py315"` only when the installed Ruff version recognizes that target. Upgrade Ruff to a compatible release if necessary. If Ruff does not yet support Python 3.15, use the newest supported target temporarily and avoid enabling syntax-specific rules that assume unsupported language features.

### Rule Reference

| Rule | Purpose |
|---|---|
| `E4`, `E7`, `E9` | Detect selected code-quality and syntax errors |
| `F` | Detect undefined names and unused imports |
| `I` | Enforce import ordering |
| `B` | Detect common bug-prone patterns |
| `UP` | Identify opportunities to modernize Python syntax |

Ruff's target version controls the Python syntax and compatibility assumptions used by its checks. It does not install Python or guarantee that third-party dependencies work on Python 3.15.

## 5. Standard Development Workflow

Run the following commands from the project root before committing:

```bash
# 1. Apply formatting
uv run ruff format .

# 2. Fix supported linting issues
uv run ruff check --fix .

# 3. Verify formatting
uv run ruff format --check .

# 4. Verify linting
uv run ruff check .

# 5. Run automated tests
uv run pytest

# 6. Check lockfile consistency
uv lock --check

# 7. Review changes
git status
git diff
```

Review all automatically modified files before committing.

## 6. CI/CD Validation

Use the same Python version and dependency lockfile in development and CI.

```bash
# Install the locked dependencies without changing uv.lock
uv sync --locked

# Validate lockfile consistency
uv lock --check

# Verify formatting
uv run --locked ruff format --check .

# Verify linting
uv run --locked ruff check .

# Run automated tests
uv run --locked pytest
```

Configure CI to select Python 3.15 explicitly. If the selected CI image or runner does not provide the required version, install it through an appropriate supported mechanism before running the checks.

For broader compatibility, test each supported Python version in a separate CI job.

## 7. Git Practices

### Files to Commit

```text
pyproject.toml
uv.lock
.python-version
README.md
```

### Files to Ignore

Add appropriate entries to `.gitignore`:

```gitignore
# Virtual environment
.venv/

# Python cache
__pycache__/
*.py[cod]

# Test and coverage cache
.pytest_cache/
.coverage
htmlcov/

# Ruff cache
.ruff_cache/

# Environment variables and secrets
.env
.env.*
!.env.example

# Build artifacts
build/
dist/
*.egg-info/
```

Never commit API keys, credentials, or production secrets.

## 8. Python 3.15+ Compatibility Practices

1. **Verify interpreter availability:** Confirm that the intended Python 3.15 release is available for your operating system and architecture.
2. **Check package compatibility:** Review the Python requirements and supported wheels for every important dependency.
3. **Keep tools current:** Use uv, Ruff, pytest, and compatible plugins that support the selected interpreter.
4. **Separate target from runtime:** Ruff's `target-version` does not determine which Python executable uv uses.
5. **Test actual behavior:** Run unit and integration tests under the intended Python version.
6. **Review native dependencies:** Packages with C or Rust extensions may need compatible wheels or updated build toolchains.
7. **Keep environments reproducible:** Commit `uv.lock` and use locked synchronization in CI.
8. **Test supported versions:** If the application supports several Python versions, run a separate test job for each.
9. **Avoid unverified language features:** Confirm that syntax and standard-library APIs exist in the exact interpreter version you deploy.
10. **Use staged adoption:** If Python 3.15 support is incomplete in your dependencies, use it for compatibility evaluation before making it the production baseline.

## 9. Official Documentation

- [uv — Project Guide](https://docs.astral.sh/uv/guides/projects/)
- [uv — Python Versions](https://docs.astral.sh/uv/guides/install-python/)
- [uv — Locking and Syncing](https://docs.astral.sh/uv/concepts/projects/sync/)
- [Ruff — Configuration](https://docs.astral.sh/ruff/configuration/)
- [Ruff — Linter Rules](https://docs.astral.sh/ruff/rules/)
- [pytest — Getting Started](https://docs.pytest.org/en/stable/getting-started.html)
- [Python — What's New](https://docs.python.org/3/whatsnew/)

---

## Quick Reference

| Task | Command |
|---|---|
| Install Python 3.15 | `uv python install 3.15` |
| Pin project Python | `uv python pin 3.15` |
| Add dependency | `uv add package-name` |
| Add dev dependency | `uv add --dev package-name` |
| Run Python | `uv run python main.py` |
| Lint | `uv run ruff check .` |
| Fix lint issues | `uv run ruff check --fix .` |
| Format | `uv run ruff format .` |
| Check formatting | `uv run ruff format --check .` |
| Run tests | `uv run pytest` |
| Inspect dependencies | `uv tree` |
| Check lockfile | `uv lock --check` |
| Synchronize locked environment | `uv sync --locked` |

**Daily rule:** Format → Lint → Test → Review diff → Commit.

Use uv for environment and dependency reproducibility, Ruff for linting and formatting, pytest for behavior verification, and CI/CD to enforce consistent standards across the team.
