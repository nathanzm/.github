# AGENTS.md

Guidance for anyone, human or AI, working in this repository.

## What this repo is
Default community health files and reusable GitHub Actions workflows for every repo owned by `nathanzm`. Other repos run this code in their CI, so a change here changes every caller's required checks. Treat it as security-critical.

## Rules
- **Security first.** Every workflow declares `permissions: contents: read` at the top and widens per job only when a job truly needs it. Checkout always uses `persist-credentials: false`.
- **Pin everything.** Third-party actions are pinned by full commit SHA, with the version in a trailing comment. Downloaded binaries are pinned by version and verified against a SHA-256 written in the workflow, never against a checksum file fetched at run time.
- **No secrets, no `pull_request_target`.** Reusable workflows take no secrets.
- **Coverage floors live here.** The 100% floor is set on the command line in `python.yml` so a calling repo can't lower it.
- **Vendor-neutral.** No AI-vendor-specific files (`CLAUDE.md`, `.claude/`, `GEMINI.md`, `.cursor/`, `.github/copilot-instructions.md`, and so on). This file is the only AI guidance.
- File names: workflows and config files are `snake_case`. Text files use LF line endings (`.gitattributes`).
- Keep docs short and in the present tense; no history narration.

## Changing a workflow
1. Run `actionlint` locally if it's available; CI runs it either way.
2. Test the change in a scratch repo that calls the workflow at the branch's commit SHA, including one run that must fail (for example 99% coverage, or a fake secret).
3. Merge, then tag a semver release (`vX.Y.Z`). Callers pin the release's commit SHA, and Dependabot proposes updates from the tags.
4. A change that adds a check or makes one stricter is a minor release; one that breaks existing callers (renamed inputs or jobs) is a major release.

## Bumping pinned tools
- Actions: Dependabot opens the PRs (7-day cooldown).
- gitleaks and actionlint: update the version and the SHA-256 together, taking the checksum from the release's official checksums file.
