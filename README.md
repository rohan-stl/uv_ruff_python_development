
# UV + Ruff — Python Development Cheat Sheet

A practical reference for Python development using **uv** for environment and dependency management, **Ruff** for linting and formatting, and **pytest** for automated testing.

## Table of Contents

- [1. Installation](#1-installation)
- [2. Create a New Project](#2-create-a-new-project)
- [3. Essential Daily Commands](#3-essential-daily-commands)
- [4. Ruff Configuration](#4-ruff-configuration)
- [5. Standard Development Workflow](#5-standard-development-workflow)
- [6. CI/CD Validation](#6-cicd-validation)
- [7. Git Practices](#7-git-practices)
- [8. Core Development Rules](#8-core-development-rules)
- [9. Official Documentation](#9-official-documentation)

---

## 1. Installation

Install `uv` on Ubuntu/Linux:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Verify the installation:

```bash
uv --version
```

Ruff should preferably be installed as a project development dependency instead of relying on a global installation.

## 2. Create a New Project

Create a Python project using Python 3.12:

```bash
# Install the Python version managed by uv
uv python install 3.12

# Create the project
uv init my-python-app
cd my-python-app

# Pin the development Python version
uv python pin 3.12

# Install development tools
uv add --dev ruff pytest

# Synchronize the environment and generate the lockfile
uv sync
```

Typical project structure:

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

The exact structure depends on the project type. Libraries, APIs, and CLI applications may use different layouts.

## 3. Essential Daily Commands

### 3.1 Dependency Management

| Command | Purpose |
|---|---|
| `uv add fastapi` | Add a runtime dependency |
| `uv add --dev ruff pytest` | Add development dependencies |
| `uv remove fastapi` | Remove a dependency |
| `uv sync` | Synchronize the environment |
| `uv tree` | Display the dependency tree |
| `uv lock` | Resolve dependencies and update the lockfile when needed |
| `uv lock --check` | Check lockfile consistency |
| `uv lock --upgrade` | Upgrade dependencies and update the lockfile |

**Best practice:** Manage project dependencies through `uv add` and `uv remove`. Commit `uv.lock` to Git to maintain reproducible dependency resolution.

### 3.2 Run Python Code

```bash
# Run a Python script
uv run python main.py

# Run a Python module
uv run python -m my_package

# Run a FastAPI application
uv run uvicorn app.main:app --reload

# Execute an inline Python command
uv run python -c "print('Hello, Python!')"
```

Prefer `uv run` for project commands because it executes them in the project's managed environment.

### 3.3 Ruff Linting

```bash
# Check the project for linting issues
uv run ruff check .

# Automatically fix supported linting issues
uv run ruff check --fix .

# Display details about a particular rule
uv run ruff rule F401
```

Use automatic fixes carefully and review the resulting changes before committing.

### 3.4 Ruff Formatting

```bash
# Format Python source files
uv run ruff format .

# Check formatting without modifying files
uv run ruff format --check .

# Preview formatting differences
uv run ruff format --diff .
```

**Rule:** Use `ruff format` to apply formatting and `ruff format --check .` to validate formatting in CI.

### 3.5 Testing with pytest

```bash
# Run all tests
uv run pytest

# Run a specific test file
uv run pytest tests/test_main.py

# Run a specific test function
uv run pytest tests/test_main.py::test_example

# Run tests with verbose output
uv run pytest -v
```

Linting and formatting do not verify application behavior. Run automated tests separately.

## 4. Ruff Configuration

Add the following configuration to `pyproject.toml`.

If these sections already exist, merge the settings rather than duplicating TOML tables.

```toml
[tool.ruff]
line-length = 100
target-version = "py312"
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

### Rule Reference

| Rule | Purpose |
|---|---|
| `E4`, `E7`, `E9` | Detect selected syntax and code-quality errors |
| `F` | Detect undefined names and unused imports |
| `I` | Enforce consistent import ordering |
| `B` | Detect common bug-prone patterns |
| `UP` | Identify opportunities to modernize Python syntax |

**Important:** `target-version = "py312"` configures Ruff's Python syntax target. It should align with the project's supported Python versions.

For example, use `py310` if Python 3.10 is the minimum supported version.

## 5. Standard Development Workflow

Run the following sequence from the project root before committing changes:

```bash
# 1. Apply consistent formatting
uv run ruff format .

# 2. Fix supported linting issues
uv run ruff check --fix .

# 3. Verify formatting
uv run ruff format --check .

# 4. Verify linting
uv run ruff check .

# 5. Run automated tests
uv run pytest

# 6. Verify lockfile consistency
uv lock --check

# 7. Review modified files
git status
git diff
```

### Recommended Development Sequence

1. Create a feature branch.
2. Implement the required changes.
3. Add or update automated tests.
4. Format the code with Ruff.
5. Fix linting issues.
6. Run tests and inspect failures.
7. Review the source-code and lockfile diffs.
8. Commit the changes.

Do not automatically fix every issue without reviewing the changes. Avoid suppressing lint rules unless there is a justified reason.

## 6. CI/CD Validation

Use the following sequence in CI pipelines to enforce reproducible dependencies, linting, formatting, and tests.

```bash
# Synchronize without allowing lockfile changes
uv sync --locked

# Validate dependency lockfile consistency
uv lock --check

# Validate formatting
uv run --locked ruff format --check .

# Validate linting
uv run --locked ruff check .

# Execute automated tests
uv run --locked pytest
```

### CI/CD Principles

- Use the committed `uv.lock` file.
- Fail the pipeline when linting or formatting checks fail.
- Fail the pipeline when automated tests fail.
- Do not automatically rewrite source files during validation.
- Keep Python and tool versions consistent across development and CI.
- Require successful checks before merging changes.

## 7. Git Practices

### Files to Commit

```text
pyproject.toml
uv.lock
.python-version
README.md
```

Commit application source code, tests, and any other required project configuration as well.

### Files to Ignore

Add appropriate entries to `.gitignore`:

```gitignore
# Python virtual environment
.venv/

# Python cache
__pycache__/
*.py[cod]

# Test cache and coverage
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

Review `.env.example` before committing it to ensure it contains placeholders rather than credentials.

### Useful Git Commands

```bash
# Check working-tree status
git status

# Review unstaged changes
git diff

# Review staged changes
git diff --cached

# Stage changes
git add .

# Commit changes
git commit -m "Add feature or fix"

# Review recent commits
git log --oneline -5
```

## 8. Core Development Rules

1. **Isolated environments:** Use one `.venv` per project.
2. **Dependency management:** Use `uv add` and `uv remove` for declared dependencies.
3. **Execution:** Prefer `uv run` for project commands.
4. **Reproducibility:** Commit `uv.lock` and review lockfile changes.
5. **Formatting:** Use Ruff to maintain consistent formatting.
6. **Linting:** Fix actionable linting issues instead of suppressing them indiscriminately.
7. **Testing:** Run automated tests independently of linting.
8. **Editor configuration:** Configure PyCharm to use the project's `.venv` interpreter.
9. **CI enforcement:** Apply the same quality checks in CI.
10. **Security:** Never commit API keys, credentials, or production secrets.
11. **Dependency updates:** Upgrade dependencies deliberately and review compatibility.
12. **Maintainability:** Add type checking and dependency security scanning as the project matures.

## 9. Official Documentation

- [uv — Project Guide](https://docs.astral.sh/uv/guides/projects/)
- [uv — Project Concepts](https://docs.astral.sh/uv/concepts/projects/)
- [uv — Locking and Syncing](https://docs.astral.sh/uv/concepts/projects/sync/)
- [Ruff — Configuration](https://docs.astral.sh/ruff/configuration/)
- [Ruff — Linter Rules](https://docs.astral.sh/ruff/rules/)
- [pytest — Getting Started](https://docs.pytest.org/en/stable/getting-started.html)

---

## Quick Reference

| Task | Command |
|---|---|
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
| Synchronize environment | `uv sync` |

**Daily rule: Format → Lint → Test → Review diff → Commit.**

Use uv for reproducible environments, Ruff for code quality, pytest for behavior verification, and CI/CD to enforce the same development standards across the team.
