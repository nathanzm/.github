# .github

Shared defaults for every repository under `nathanzm`:

- **Security policy** (`SECURITY.md`) and a **contributing notice** (`CONTRIBUTING.md`: these repos aren't intended for outside contributions), shown on every repo that has none of its own.
- **One reusable CI workflow, `standards.yml`**, that every repo calls. It detects the languages in the repo and runs each one's gate:

| Job | Runs | Checks |
|---|---|---|
| `detect` | Always | Finds the languages; fails on code in a language with no gate here |
| `secrets` | Always | gitleaks over the full history of the checked-out commit |
| `vendor-files` | Always | No AI-vendor-specific files |
| `dependency-review` | PRs to public repos | Vulnerable dependencies added by the PR |
| `actionlint` | Workflows present | actionlint (with shellcheck) over every workflow |
| `python` | Python present | ruff format and lint, mypy strict, pytest with a 100% line and branch coverage floor |
| `result` | Always | The one required check: every gate that should run passed, and the rest were skipped |

## Use

Call it from `.github/workflows/ci.yml` at `@main`, and require only `standards / result` on `main`:

```yaml
jobs:
  standards:
    uses: nathanzm/.github/.github/workflows/standards.yml@main
    # with:
    #   exclude: vendor/ third_party/   # path prefixes detection ignores
```

`nathanzm/repo-template` includes this file. Adding a language turns on its gate with no change to the caller or its required check. Code in a language with no gate fails `detect` until a gate is added here. Changes here reach every caller as soon as they merge, so this repo is the one place where third-party actions and tools are pinned; callers pin nothing.

## Python setup

The `python` gate runs in the repo root and expects these files there.

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
