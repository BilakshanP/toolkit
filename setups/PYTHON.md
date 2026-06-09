# Python Project Setup Guide (for LLM)

Instructions for setting up a Python project with uv, hatchling, basedpyright, and ruff. Adapt names/versions to the target project.

## Install uv

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Directory layout

```
project-root/
├── src/
│   └── <package_name>/
│       ├── __init__.py
│       ├── __main__.py        # optional: enables `python -m <package_name>`
│       ├── cli.py             # if CLI tool: argparse entry point
│       └── ...
├── tests/
│   ├── conftest.py            # shared fixtures
│   ├── fixtures/              # test data files (optional)
│   └── test_*.py              # pytest tests
├── .githooks/
│   └── pre-commit             # ruff + basedpyright checks
├── pyproject.toml
├── pyrightconfig.json
├── .python-version
├── .gitignore
└── .github/workflows/typecheck.yml
```

Uses the `src/` layout — all package code lives under `src/<package_name>/`.

## .python-version

Pin the Python version. uv reads this to provision the interpreter.

```
3.14
```

## pyproject.toml

```toml
[project]
name = "<project-name>"
version = "0.1.0"
description = "<one-line description>"
requires-python = ">=3.14"
dependencies = []

[project.scripts]
<cli-name> = "<package_name>.cli:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[dependency-groups]
dev = [
    "basedpyright>=1.39.2",
    "ruff>=0.15.11",
    "pytest>=7.0",
    "pytest-cov>=4.0",
]
```

Key points:
- **hatchling** as build backend. It auto-discovers the `src/` layout and includes non-`.py` files (`.css`, `.js`, etc.) by default — no custom build config needed.
- **No runtime dependencies.** Add them under `dependencies = [...]` if needed.
- **Dev tools** go in `[dependency-groups] dev`, not in `dependencies`.
- **`[project.scripts]`** creates the CLI entry point. The function must be importable as `<package_name>.cli:main`. Remove this section if there's no CLI.

Add pytest configuration:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
```

## pyrightconfig.json

```json
{
  "include": ["src"],
  "venvPath": ".",
  "venv": ".venv",
  "typeCheckingMode": "strict"
}
```

`"include": ["src"]` tells basedpyright where to find the source. `venvPath`/`venv` point it at the uv-managed virtualenv.

## .gitignore

```
# Python-generated files
__pycache__/
*.py[oc]
build/
dist/
wheels/
*.egg-info

# Virtual environments
.venv
```

Add project-specific patterns (generated output files, fixture dirs, etc.) as needed.

## GitHub Actions CI (.github/workflows/typecheck.yml)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - uses: astral-sh/setup-uv@v6

      - run: uv sync --dev

      - name: Format & lint fix
        run: |
          uv run ruff format src/
          uv run ruff check --fix src/

      - name: Commit formatting changes
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "Formatted: code with ruff"

      - run: uv run basedpyright

      - name: Test
        run: uv run pytest --cov
```

This auto-formats and auto-fixes lint issues on push, commits the result, then runs the type checker and tests. If any step fails, the workflow fails.

## Pre-commit hooks (.githooks/pre-commit)

```bash
#!/bin/sh
VENV=".venv/bin"
$VENV/ruff check src/ > /dev/null 2>&1 || { echo "pre-commit: ruff check failed (run 'ruff check src/' for details)"; exit 1; }
$VENV/ruff format --check src/ > /dev/null 2>&1 || { echo "pre-commit: ruff format failed (run 'ruff format src/')"; exit 1; }
$VENV/basedpyright src/ > /dev/null 2>&1 || { echo "pre-commit: basedpyright failed (run 'basedpyright src/' for details)"; exit 1; }
```

Make it executable:

```bash
chmod +x .githooks/pre-commit
```

Configure git to use `.githooks/` as the hooks directory (instead of the default `.git/hooks/`):

```bash
git config core.hooksPath .githooks
```

This is part of the initial setup — include it in the README/CONTRIBUTING so contributors run it after cloning.

## Dev workflow commands

```bash
# Initial setup (creates .venv, installs dev deps, enables hooks)
uv sync --dev
git config core.hooksPath .githooks

# Type check
uv run basedpyright

# Lint
uv run ruff check src/

# Format
uv run ruff format src/

# Check format without modifying
uv run ruff format --check src/

# Run tests
uv run pytest

# Run tests with coverage
uv run pytest --cov

# Run a specific test
uv run pytest tests/test_foo.py -v

# Run the CLI during development
uv run <cli-name> [args...]

# One-off scripts with temp dependencies
uv run --with rich script.py

# Run CLI tools without installing
uvx ruff check .

# Pin Python version
uv python install 3.14
uv python pin 3.14
```

## Coding conventions to keep basedpyright / ruff clean

- Extract `argparse.Namespace` fields via `typing.cast` at the top of the consuming function — avoids `reportAny` on `args.X` access.
- Prefix unused return values with `_ =` (e.g., `_ = parser.add_argument(...)`, `_ = f.write(...)`, `_ = items.pop()`).
- Don't use adjacent implicit string concatenation — use explicit `+` between string literals / f-strings.
- Type `re.findall()` results explicitly when the pattern has groups (returns `list[tuple[...]]`).
- Only use `f"..."` when the string actually interpolates — ruff's `F541` flags bare f-string literals with no placeholders.
- Run `ruff format` before committing; follow its default style.

## Enabling `python -m <package_name>`

Create `src/<package_name>/__main__.py`:

```python
from <package_name>.cli import main

main()
```

## Including static assets

If the package ships non-Python files (CSS, JS, templates, etc.), put them in a sub-package with an `__init__.py`:

```
src/<package_name>/assets/
├── __init__.py    # empty, makes it a package
├── styles.css
└── script.js
```

Load at import time via:

```python
from importlib.resources import files

CSS = files("<package_name>.assets").joinpath("styles.css").read_text()
JS = files("<package_name>.assets").joinpath("script.js").read_text()
```

hatchling includes these files automatically — no extra config.
