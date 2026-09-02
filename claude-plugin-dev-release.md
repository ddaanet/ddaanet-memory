---
name: claude-plugin-dev-release
description: "`error: uncommitted changes` = real work (only `.claude` and the memory mount are exempt); a release that dies at the marketplace bump has already published commit, tag, push and GitHub release, so `just resume-release` finishes it and never retries from the top; the marketplace push needs `/add-dir` on that sibling repo; manifest holds the LAST released version and a never-released plugin takes no bump argument; `edify` ports the flow locally for its PyPI step"
metadata:
  node_type: memory
  type: project
---
Running a release in a Claude Code plugin that vendors the `claude-plugin-dev`
toolkit at `plugin-dev/`. Vendoring, installing and updating the toolkit is
[[claude-plugin-dev]].

**Why:** the release is one sequence — version commit, tag, branch push,
GitHub release, marketplace bump — landing publicly in that order, so a
failure partway through leaves a half-published release, not nothing.

**How to apply:**

- **`just resume-release` finishes a half-done release, and is idempotent** —
  every step probes what already landed before acting. The marketplace bump is
  the recipe's last step and its only write outside the repo, so a run that
  dies there has already made the version commit, tag, branch push and GitHub
  release public. The recovery is to finish it, never to retry from the top.
- The marketplace lives in a sibling repo, and the push into it is refused by
  the auto-mode classifier as an *external repo outside the trusted source
  control org* until my human partner has run `/add-dir` on that repo
  ([[classifier-denied-self-config]]).
- **`error: uncommitted changes` names real work, with two exemptions.**
  `release.sh`'s `tree_is_clean` gates on `git diff --quiet HEAD` with two
  pathspec exclusions: `.claude` (agent working state — the `handoff` and
  `precompact` frames staged for "whatever commit lands next", `settings.json`,
  `gitlore-memory-message`) and the gitlore memory mount path, read from
  `.gitmodules` by the submodule name `gitlore-memory`. `.claude-plugin` is NOT
  excluded — git pathspecs match at the path separator — so a dirty manifest
  still refuses. Anything staged under `.claude` rides the release commit.
  The same check runs against `MARKETPLACE_DIR`, where an unrelated dirty file
  fails the release at its last step. A consumer whose vendored `plugin-dev/`
  predates the `.claude` exclusion still refuses on a staged frame; read its
  `tree_is_clean` before blaming the tree.
- Plugin manifest holds the *last released* version; `just release
  {patch|minor|major}` bumps from there. Manual edits are blocked by the
  version-guard hook and the recipe's pre-flight check. A plugin that has
  never been released has nothing to bump from: its first `just release`
  takes no bump argument and publishes whatever version `plugin.json`
  already holds; passing a bump on that first call is refused.
- Release commit message convention: `release: X.Y.Z` (gitmoji hook
  rewrites to `🔖 release X.Y.Z`).
- Default branch is auto-detected from `origin/HEAD` — recipes don't
  hardcode `main`.
- `edify` doesn't vendor the toolkit: hybrid Python+plugin shape with a PyPI
  publish the toolkit's github+marketplace flow doesn't cover. It carries a
  local `scripts/release.sh`, ported from this toolkit's `release.sh` —
  `plugin.json` as version SOT, the same per-step probe-before-act shape for
  `--resume` — with `pyproject.toml` synced from `plugin.json` (never the
  reverse) and a PyPI-publish step (checks `pypi.org/pypi/<name>/<version>/json`
  before building) spliced between the tag push and the GitHub release. It sits
  at the repo root: `plugin/` ships verbatim to every install, so
  MARKETPLACE_DIR paths and PyPI credentials have no business inside it.
