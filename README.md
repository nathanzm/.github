# .github

Shared defaults for every repository under `nathanzm`:

- **Security policy and contributing guide** (`SECURITY.md`, `CONTRIBUTING.md`), shown on every public repo that has none of its own.
- **Reusable CI workflows** that other repos call:

| Workflow | Checks |
|---|---|
| `baseline.yml` | gitleaks over the full history, a ban on AI-vendor-specific files, dependency review on public PRs |
| `python.yml` | ruff format and lint, mypy, pytest with a 100% line and branch coverage floor |

## Use

Call the workflows from your repo's CI, pinned to a full commit SHA:

```yaml
jobs:
  baseline:
    uses: nathanzm/.github/.github/workflows/baseline.yml@<commit-sha> # v1.0.0
  python:
    uses: nathanzm/.github/.github/workflows/python.yml@<commit-sha> # v1.0.0
```

`nathanzm/repo-template` already includes this setup.
