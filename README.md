# Atlas-X-AI / .github

Org-level defaults and reusable workflows for Atlas-X-AI repositories.

## Reusable CI workflow

File: [`.github/workflows/atlas-ci.yml`](.github/workflows/atlas-ci.yml)

### How to use in a new repo

Drop this file into `<repo>/.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  ci:
    uses: Atlas-X-AI/.github/.github/workflows/atlas-ci.yml@main
    secrets: inherit
```

That's it. Every PR and every push to `main` will run:

1. **gitleaks** — fast pattern-based secret scan (blocking)
2. **trufflehog** — deep verified-secret scan (blocking)
3. **pytest** via `uv` — only if `pyproject.toml` + `tests/` exist (auto-detected)
4. **ruff** lint — only if Python project (non-blocking warnings for now)

### Inputs (optional)

```yaml
jobs:
  ci:
    uses: Atlas-X-AI/.github/.github/workflows/atlas-ci.yml@main
    with:
      python-version: "3.11"   # default 3.12
      skip-tests: true         # for docs-only repos
    secrets: inherit
```

## Branch protection pattern

After wiring up `ci.yml`, enable branch protection on `main` via repo Settings → Rules → Rulesets (or via `gh api`):

- Require a pull request before merging
- Require status checks to pass: `Secret scan (gitleaks)`, `Secret scan (trufflehog)`, `Python tests (pytest via uv)` (if Python)
- Require branches to be up to date before merging
- Do not allow bypassing the above

## PR Evidence Contract

Files:

- [`pull_request_template.md`](pull_request_template.md)
- [`.github/workflows/pr-evidence-contract.yml`](.github/workflows/pr-evidence-contract.yml)

The template provides the default PR body shape across Atlas-X-AI repositories. The workflow enforces the same contract by validating PR bodies, adding a bot comment when context is missing, failing the run, and converting noncompliant PRs back to draft.

To enforce it in a repo, add this caller workflow:

```yaml
name: PR Evidence Contract

on:
  pull_request_target:
    types: [opened, edited, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  issues: write
  pull-requests: write

jobs:
  pr-evidence-contract:
    uses: Atlas-X-AI/.github/.github/workflows/pr-evidence-contract.yml@main
    permissions:
      contents: read
      issues: write
      pull-requests: write
```

## Going-public ceremony

When a private Atlas-X-AI repo graduates to public release: see [`docs/going-public.md`](docs/going-public.md).

Short version: squash all history into one orphan commit, run scanners against the orphan branch, push that one commit to a new public repo. No in-progress mistakes ever hit the public commit log.

## Why this repo exists

- **Single source of truth** for org-wide CI and PR governance. Improvements here apply to every repo automatically.
- **`.github` is a special repo name** — GitHub recognizes it as the org-level defaults location.
- **Public by design.** Default pull request templates require a public `.github` repo, and public reusable workflows can be called by both public and private repositories in the same owner space.
