---
name: python-project-setup
description: >-
  Scaffold and configure modern Python 3.10+ projects using uv for package
  management and virtual environments. Use when the user asks to create a new
  Python project, set up a Python environment, initialize pyproject.toml,
  configure dependencies, or scaffold a Python application structure.
---

# Python Project Setup with uv

Set up modern Python 3.10+ projects using **uv** as the package manager and virtual environment tool.

## Prerequisites

Verify uv is installed before proceeding:

```bash
uv --version
```

If not installed, install via:

```bash
# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Project Initialization

### New project from scratch

```bash
uv init project-name
cd project-name
```

This creates:
- `pyproject.toml` — project metadata and dependencies
- `src/project_name/` — source package (src layout)
- `.python-version` — pinned Python version
- `README.md`

### For an existing directory

```bash
cd existing-project
uv init
```

## Standard Directory Structure

```
project-name/
├── pyproject.toml
├── uv.lock
├── .python-version
├── .gitignore
├── README.md
├── src/
│   └── project_name/
│       ├── __init__.py
│       └── main.py
├── tests/
│   ├── __init__.py
│   └── test_main.py
└── .github/
    └── workflows/
        └── ci.yml
```

Always use the **src layout** (`src/package_name/`) over flat layout for proper isolation during testing.

## pyproject.toml Configuration

Minimal `pyproject.toml` for a modern Python project:

```toml
[project]
name = "project-name"
version = "0.1.0"
description = "Project description"
requires-python = ">=3.10"
dependencies = []

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=5.0",
    "ruff>=0.4",
]

[tool.ruff]
target-version = "py310"
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP", "S", "B", "A", "C4", "PT"]

[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["src"]
```

### Key rules for pyproject.toml

- Always set `requires-python = ">=3.10"`
- Group dev tools under `[project.optional-dependencies] dev`
- Use ruff for both linting and formatting (replaces black, isort, flake8)
- Enable security-related ruff rules (`S` = bandit equivalent)

## Virtual Environment & Dependencies

```bash
# Create venv and install project with dev deps
uv venv
uv pip install -e ".[dev]"

# Add a runtime dependency
uv add requests

# Add a dev dependency
uv add --dev pytest-asyncio

# Remove a dependency
uv remove requests

# Sync environment from lock file
uv sync
```

### Lock file

Always commit `uv.lock` to version control. It ensures reproducible installs across environments.

## Running Commands

Use `uv run` to execute within the project environment without manual activation:

```bash
uv run python src/project_name/main.py
uv run pytest
uv run ruff check .
uv run ruff format .
```

## CI/CD Configuration

### GitHub Actions

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12", "3.13"]

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Set up Python ${{ matrix.python-version }}
        run: uv python install ${{ matrix.python-version }}

      - name: Install dependencies
        run: uv sync --all-extras

      - name: Lint
        run: uv run ruff check .

      - name: Format check
        run: uv run ruff format --check .

      - name: Test
        run: uv run pytest --cov=src --cov-report=xml

      - name: Upload coverage
        if: matrix.python-version == '3.12'
        uses: codecov/codecov-action@v4
        with:
          file: coverage.xml
```

### GitLab CI

Create `.gitlab-ci.yml`:

```yaml
stages:
  - lint
  - test

variables:
  UV_CACHE_DIR: .uv-cache

cache:
  paths:
    - .uv-cache

.uv-setup: &uv-setup
  before_script:
    - curl -LsSf https://astral.sh/uv/install.sh | sh
    - export PATH="$HOME/.local/bin:$PATH"
    - uv sync --all-extras

lint:
  stage: lint
  image: python:3.12-slim
  <<: *uv-setup
  script:
    - uv run ruff check .
    - uv run ruff format --check .

test:
  stage: test
  image: python:3.12-slim
  <<: *uv-setup
  script:
    - uv run pytest --cov=src --cov-report=term
  parallel:
    matrix:
      - PYTHON_VERSION: ["3.10", "3.11", "3.12", "3.13"]
```

## .gitignore Essentials

Ensure `.gitignore` includes:

```gitignore
__pycache__/
*.py[cod]
*$py.class
*.egg-info/
dist/
build/
.venv/
.env
.ruff_cache/
.pytest_cache/
.coverage
coverage.xml
htmlcov/
```

## Modern Python Patterns

When writing code in the scaffolded project, prefer:

- `from __future__ import annotations` at the top of every module
- Type hints on all public function signatures
- `pathlib.Path` over `os.path`
- `dataclasses` or `pydantic` for data containers
- `match` statements (3.10+) where appropriate
- `tomllib` (3.11+) for reading TOML config; `tomli` as fallback for 3.10

## Quick Reference

| Task | Command |
|------|---------|
| Create project | `uv init project-name` |
| Create venv | `uv venv` |
| Add dependency | `uv add package-name` |
| Add dev dependency | `uv add --dev package-name` |
| Install all deps | `uv sync` |
| Run script | `uv run python script.py` |
| Run tests | `uv run pytest` |
| Lint | `uv run ruff check .` |
| Format | `uv run ruff format .` |
| Pin Python version | `uv python pin 3.12` |
