---
name: edify-release-flow
description: "`edify` does not vendor the claude-plugin-dev toolkit: a local root `scripts/release.sh` ported from it, `pyproject.toml` synced FROM `plugin.json` and never the reverse, PyPI publish spliced between the tag push and the GitHub release"
metadata: 
  node_type: memory
  type: project
  originSessionId: 65f754d8-cdbe-42af-8461-7c22a89d764a
  modified: 2026-09-03T08:47:32.290Z
---

Cutting a release in `edify`, the one repo among my human partner's plugins that
is deliberately out of the toolkit's scope. Vendoring the toolkit into the
plugins that *do* use it is [[claude-plugin-dev]].

**Why:** edify is a hybrid Python package plus Claude Code plugin. The toolkit's
flow ends at a GitHub release and a marketplace bump and has no PyPI step, so
wrapping edify would either need conditionals that obscure the main path or
break it outright. That exclusion is a stated limitation of the toolkit's
design, not an oversight.

**How to apply:**

- edify carries a local `scripts/release.sh`, ported from the toolkit's
  `release.sh`: `plugin.json` is the version source of truth, and `--resume`
  keeps the same per-step probe-before-act shape.
- `pyproject.toml` is synced *from* `plugin.json`, never the reverse.
- The PyPI publish is spliced between the tag push and the GitHub release, and
  checks `pypi.org/pypi/<name>/<version>/json` before building.
- The script sits at the repo root, not under `plugin/`: that directory ships
  verbatim to every install, so MARKETPLACE_DIR paths and PyPI credentials have
  no business inside it.
