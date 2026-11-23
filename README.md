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

`nathanzm/repo-template` already includes this setup. Changes here reach every caller as soon as they merge, so this repo is the one place where third-party actions and tools are pinned; callers pin nothing.
