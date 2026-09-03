---
name: claude-plugin-dev
description: "shared release infra vendored at a `dist-vX.Y.Z` tag into `plugin-dev/`; the procedure is its own `toolkit/README.md` and the failure modes are `plugin-craft:toolkit-release`; `edify` ports `release.sh` locally instead of vendoring, for the PyPI step"
metadata: 
  node_type: memory
  type: project
  originSessionId: 194d9f85-63c2-4d23-8e81-c87672ce360c
  modified: 2026-09-01T18:16:52.594Z
---
`claude-plugin-dev` (https://github.com/ddaanet/claude-plugin-dev) is the
canonical release infrastructure for my human partner's Claude Code plugins
(`handoff`, `gitmoji`, and others), vendored per-consumer via `git subtree`
from a `toolkit/` prefix in the source repo into `plugin-dev/` in the plugin.

Two documents cover using it, and neither is repeated here. `toolkit/README.md`
ships with the vendored files and states what each is for, the two-tag
`vX.Y.Z` / `dist-vX.Y.Z` scheme, installing, updating, the required `precommit`
and `prerelease` recipes, and the release conventions. What goes wrong while
running it — the sandbox and classifier failures that leave a release
half-landed, what `error: uncommitted changes` actually excludes, the
`just --list` check owed after a subtree pull — is the
`plugin-craft:toolkit-release` skill.

One consumer is an exception:

- `edify` doesn't vendor the toolkit: hybrid Python+plugin shape with a PyPI
  publish the toolkit's github+marketplace flow doesn't cover. It carries a
  local `scripts/release.sh`, ported from this toolkit's `release.sh` —
  `plugin.json` as version SOT, the same per-step probe-before-act shape for
  `--resume` — with `pyproject.toml` synced from `plugin.json` (never the
  reverse) and a PyPI-publish step (checks `pypi.org/pypi/<name>/<version>/json`
  before building) spliced between the tag push and the GitHub release.
  Release tooling belongs at the repo root: `plugin/` ships verbatim to every
  plugin install, so MARKETPLACE_DIR paths and PyPI credentials have no
  business living inside it.
