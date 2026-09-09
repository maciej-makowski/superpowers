# Merging upstream into this fork (keeping the Hermes install viable)

This fork exists solely so `hermes plugins install maciej-makowski/superpowers --enable`
passes Hermes' plugin security scanner. The scanner walks the entire clone, so any
regenerated `tests/`, `docs/superpowers/`, `.opencode/`, `docs/plans/` or
`RELEASE-NOTES.md` content from upstream will re-trigger false-positive findings
(fake test tokens, planning prose, etc.) and block the install again.

## One-time setup

```bash
git remote add upstream https://github.com/obra/superpowers.git
git fetch upstream
```

## After each upstream release

```bash
git fetch upstream
git checkout main
git merge upstream/main        # fast-forward usually; resolve conflicts in kept files only

# Re-apply the prune (this is the step people forget):
git rm -rq tests docs/superpowers .opencode docs/plans
git rm -f RELEASE-NOTES.md
git commit -m "Re-prune scanner-triggering paths after upstream merge"

git push origin main
```

Then update the plugin: `hermes plugins update superpowers` (or re-run install).

## Gotchas

- **Never keep those paths on `main`** — a merge that reintroduces them makes the next
  install/update fail with a `dangerous` verdict that `--force` cannot override.
- **New top-level docs may also trip the scanner** (e.g. a future `RELEASE-NOTES.md`
  replacement, or new `docs/` subdirectories with plan/spec files). If an install is
  blocked, read the findings list, prune the offending paths, and re-push.
- The prune commit should always sit **on top of** the upstream merge; don't rewrite
  history on `main` (branch protection requires squash merges with approval, which keeps
  this clean).
- Upstream occasionally renames/moves these directories (e.g. plans moving between
  `docs/plans/` and `docs/superpowers/plans/`). Check what the merge actually brought in:

  ```bash
  git diff --stat HEAD upstream/main -- . ':!skills' ':!hooks' ':!scripts' ':!.hermes-plugin'
  ```

- Keep `upstream` set in remotes so future merges are one command.
