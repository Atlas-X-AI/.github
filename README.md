# Atlas-X-AI / .github

Org-level configuration repo. Contains the **reusable CI workflow** every Atlas-X-AI repo calls, plus the "going public" ceremony docs.

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

## Going-public ceremony

When a private Atlas-X-AI repo graduates to public release: see [`docs/going-public.md`](docs/going-public.md).

Short version: squash all history into one orphan commit, run scanners against the orphan branch, push that one commit to a new public repo. No in-progress mistakes ever hit the public commit log.

## Why this repo exists

- **Single source of truth** for org-wide CI. Improvements here apply to every repo automatically.
- **`.github` is a special repo name** — GitHub recognizes it as the org-level defaults location.
- **Private by design.** Reusable workflows in a private `.github` repo can be called by other private repos in the same org, provided org Settings → Actions → General → "Allow access to reusable workflows from private repositories" is enabled (one-time toggle).
