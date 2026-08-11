---
name: claude-plugin-dev
description: "shared release infra for ddaa's plugins; VERSION file SOT, self-release recipe, marketplace bump; `just release` must be run unsandboxed or it dies at that bump on `mv: inter-device move failed`/`Read-only file system`, leaving a half-done release `resume-release` finishes"
metadata: 
  node_type: memory
  type: project
  originSessionId: 194d9f85-63c2-4d23-8e81-c87672ce360c
  modified: 2026-08-11T20:05:19.388Z
---
`claude-plugin-dev` (https://github.com/ddaanet/claude-plugin-dev) is the
canonical release infrastructure for the user's Claude Code plugins
(`handoff`, `gitmoji`, and others). Repo layout: `release.just` (thin
recipe wrappers consumers import), `release.sh` (the release flow itself,
including `--resume`), `version-guard.sh` (PreToolUse hook),
`check-version.sh` (plugin.json vs marketplace.json drift check, `just
check-version` + auto pre-flight in `release`), `install.sh` (one-shot
vendor + wire), `tests/` (hook, release and update-plugin-dev scenario
suites), `justfile` (toolkit's own `precommit` AND self-`release` recipe),
`VERSION` (plain text, last-released version), `docs/design.md`,
`docs/changelog.md` (index + one dated file per change), `CLAUDE.md`.

**Why:** Several plugins had diverged near-identical release recipes,
and an agent-driven manual edit to `plugin.json` desynced the
manifest from the latest tag — caught only at release time. Toolkit
unifies the recipe, adds a version-guard hook, and folds the marketplace
bump into the same atomic release.

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
  `just update-plugin-dev vX.Y.Z`. The recipe verifies
  `plugin-dev/VERSION` matches the requested ref after pull.
- **Toolkit self-release**: `just release` in the toolkit's own
  `justfile` bumps `VERSION`, commits, tags, pushes, and creates a
  GitHub release.
- `VERSION` exists because subtree pulls don't propagate tags —
  consumers had no way to identify the toolkit version they vendored.
  See toolkit `docs/design.md` "Toolkit version source of truth".
- **Run `just release` with the command sandbox disabled.** The
  marketplace bump is the recipe's last step and its only write outside
  the repo, so under the sandbox it dies there on `mv: inter-device move
  failed … unable to remove target: Read-only file system` against
  `$MARKETPLACE_DIR`. Everything before it — version commit, tag, branch
  push, GitHub release — has already landed, so a sandboxed run leaves a
  half-done release rather than a failed one; `just resume-release`,
  likewise unsandboxed, completes it and is idempotent. See
  [[sandbox-effects]].
- `edify` is deliberately out of scope (hybrid Python+plugin shape
  with PyPI publish; keeps its bespoke recipe).
