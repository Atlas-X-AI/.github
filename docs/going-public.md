# Going-public ceremony

When a private Atlas-X-AI repo earns the right to go public, follow this exact sequence. It makes the public history start from one clean commit with no embarrassing mid-development state, secrets, or dead-end experiments.

## Pre-flight gates (all must be true)

- [ ] All tests pass on `main` for the last 3 consecutive CI runs
- [ ] At least 3 people (or 3 fresh VMs) have successfully used the project privately
- [ ] No open critical issues or known broken features
- [ ] README, LICENSE, CONTRIBUTING (optional) are written
- [ ] You (Atlas-X-AI owner) have deliberately decided to publish — this is not a reflex

## The 5-step ceremony

All commands run from the private repo's checkout.

### 1. Create a clean orphan branch

```bash
git checkout --orphan public-release
git add -A
git commit -m "Initial public release (v<VERSION>)"
```

The `--orphan` flag creates a branch with no history. Your single commit is the genesis of the public repo.

### 2. Scan the orphan for secrets

```bash
gitleaks detect --source . --report-path /tmp/gitleaks.json --report-format json
trufflehog filesystem . --no-update --fail
```

If either finds anything, **stop**. Fix the issue on the orphan branch, re-commit, re-scan. Do not proceed with any finding, even if you "know it's fine."

### 3. Create the public repo

```bash
gh repo create Atlas-X-AI/<project>-public --public \
  --description "<one-line description>" \
  --homepage "<URL or omit>"
```

Name convention: `<project>-public` keeps the public repo distinct from the private source. Some prefer just `<project>` in a different org; pick whatever is consistent for Atlas-X-AI.

### 4. Push the orphan to the public repo

```bash
git remote add public https://github.com/Atlas-X-AI/<project>-public.git
git push public public-release:main
```

Verify via `gh repo view Atlas-X-AI/<project>-public --web` — should show one commit.

### 5. Enable branch protection + CI on the public repo

Add the caller workflow (`.github/workflows/ci.yml`) on the public side too, and enable the same branch protection rules as the private side. The public repo is not exempt from the CI bar.

## Post-ceremony

- Update the private repo's README to note: "Public mirror: `Atlas-X-AI/<project>-public`. Synced via ceremony — see [`going-public.md`](…)."
- Announce the public release via your usual channels (X, LinkedIn, Discord, newsletter).
- Going forward: if you want to push future updates to the public repo, repeat the orphan ceremony per release — do NOT `git remote add public` on your ongoing private main. Keeping them isolated prevents accidental leakage.

## What you are NOT doing

- **Not rebasing or rewriting the private repo's main.** The private history is preserved untouched.
- **Not setting up ongoing sync automation.** Per release, you do this ceremony deliberately. Automation can come later if the manual cost gets annoying (see `Atlas-X-AI/.github/docs/auto-mirror.md` if/when that exists).
- **Not making the private repo public.** The private repo stays private forever.
