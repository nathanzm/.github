# .github

Shared defaults for every repository under `nathanzm`:

- **Security policy** (`SECURITY.md`) and a **contributing notice** (`CONTRIBUTING.md`: these repos aren't intended for outside contributions), shown on every repo that has none of its own.
- **Reusable CI workflows** that other repos call:

| Workflow | Checks |
|---|---|
| `baseline.yml` | gitleaks over the full history, a ban on AI-vendor-specific files, dependency review on public PRs |
| `python.yml` | ruff format and lint, mypy, pytest with a 100% line and branch coverage floor |
| `actionlint.yml` | actionlint (with shellcheck) over every workflow in the calling repo |

## Use

Call the workflows from your repo's CI at `@main`:

```yaml
jobs:
  baseline:
    uses: nathanzm/.github/.github/workflows/baseline.yml@main
  python:
    uses: nathanzm/.github/.github/workflows/python.yml@main
```

`nathanzm/repo-template` includes the `baseline` job; add a job for each language the repo uses. Changes here reach every caller as soon as they merge, so this repo is the one place where third-party actions and tools are pinned; callers pin nothing.

## Python setup

`python.yml` runs in the caller's repo root (or `working-directory`) and expects these files there.

`pyproject.toml` (rename `example`; code in `src/<package>/`, tests in `tests/`):

```toml
[project]
name = "example"
version = "0.1.0"
description = "TODO: one-line description."
requires-python = ">=3.14"

[tool.ruff]
target-version = "py314"

[tool.ruff.lint]
# pycodestyle, pyflakes, isort, bugbear, pyupgrade, bandit security rules,
# comprehensions, simplify, ruff-specific.
select = ["E", "W", "F", "I", "B", "UP", "S", "C4", "SIM", "RUF"]

[tool.ruff.lint.per-file-ignores]
# pytest uses plain assert.
"tests/**" = ["S101"]

[tool.mypy]
strict = true
files = ["src", "tests"]
mypy_path = "src"

[tool.pytest.ini_options]
pythonpath = ["src"]
testpaths = ["tests"]
addopts = "--strict-markers"
markers = ["integration: uses real external dependencies (files, processes, services)"]

[tool.coverage.run]
source = ["src"]
branch = true
```

`requirements-dev.txt` (pinned; Dependabot keeps it current):

```text
mypy==2.4.1
pytest==9.2.0
pytest-cov==7.1.0
ruff==0.17.6
```

`.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: pip
    directory: /
    schedule:
      interval: weekly
    cooldown:
      default-days: 7
```

`.gitignore` additions: `__pycache__/`, `*.py[cod]`, `.venv/`, `build/`, `dist/`, `*.egg-info/`, `.coverage`, `.coverage.*`, `htmlcov/`, `.mypy_cache/`, `.pytest_cache/`, `.ruff_cache/`.

Local checks, as CI runs them:

```sh
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements-dev.txt
ruff format && ruff check && mypy
pytest --cov --cov-branch --cov-report=term-missing --cov-fail-under=100
```
