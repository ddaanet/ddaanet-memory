---
name: claude-plugin-dev toolkit
description: Shared release infrastructure for ddaanet's Claude Code plugins, vendored via git subtree at plugin-dev/. Self-versioned via VERSION file. Latest tag v0.2.0.
type: project
originSessionId: 194d9f85-63c2-4d23-8e81-c87672ce360c
---
`claude-plugin-dev` (https://github.com/ddaanet/claude-plugin-dev,
latest tag `v0.2.0` shipped 2026-04-29) is the canonical release
infrastructure for the user's Claude Code plugins (`handoff`,
`gitmoji`, eventually others). Repo layout: `release.just` (consumer
release recipe), `version-guard.sh` (PreToolUse hook), `install.sh`
(one-shot vendor + wire), `tests/hook-test.sh` (version-guard
scenarios), `justfile` (toolkit's own `precommit` AND self-`release`
recipe), `VERSION` (plain text, last-released version),
`DESIGN.md`, `CLAUDE.md`.

**Why:** Several plugins had diverged near-identical release recipes,
and an agent-driven manual edit to `plugin.json` desynced the
manifest from the latest tag — caught only at release time. Toolkit
unifies the recipe, adds a version-guard hook, and (since v0.2.0)
folds the marketplace bump into the same atomic release.

**How to apply:**

- Consumer plugins vendor at `plugin-dev/` via
  `git subtree add --prefix=plugin-dev <url> vX.Y.Z --squash`.
- Run `bash plugin-dev/install.sh` (or `bash /tmp/cpd/install.sh
  vX.Y.Z` first time) to wire `import 'plugin-dev/release.just'`
  into the consumer's `justfile` and the version-guard hook into
  `.claude/settings.json`.
- Consumer must define a `precommit` recipe (NOT `validate`) — the
  imported `release` depends on it.
- Consumer must export `MARKETPLACE_DIR` (typically via `.envrc`) so
  the release recipe can bump the matching entry in the
  `claude-plugins` marketplace repo and push it.
- Release commit message convention: `release: X.Y.Z` (gitmoji hook
  rewrites to `🔖 release X.Y.Z`).
- Plugin manifest version reflects the *last released* version;
  `just release {patch|minor|major}` bumps from there. Manual edits
  are blocked by the version-guard hook and the recipe's pre-flight
  check.
- Update toolkit version in a consumer with
  `just update-plugin-dev vX.Y.Z`. As of v0.2.0 the recipe verifies
  `plugin-dev/VERSION` matches the requested ref after pull.
- **Toolkit self-release** (since v0.2.0): `just release` in the
  toolkit's own `justfile` bumps `VERSION`, commits, tags, pushes,
  and creates a GitHub release. No more manual `git tag` dance.
- `VERSION` exists because subtree pulls don't propagate tags —
  consumers had no way to identify the toolkit version they vendored.
  See toolkit `DESIGN.md` "Toolkit version source of truth".
- `edify` is deliberately out of scope (hybrid Python+plugin shape
  with PyPI publish; keeps its bespoke recipe).
