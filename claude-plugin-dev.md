---
name: claude-plugin-dev
description: "shared release infra for ddaa's plugins; VERSION file SOT, self-release recipe, marketplace bump; `just release` must be run unsandboxed or it dies at that bump on `mv: inter-device move failed`/`Read-only file system`, leaving a half-done release `resume-release` finishes; `update-plugin-dev`'s subtree pull collides with a consumer's own gitlore `memory` submodule ('not our ref' fetch error) — fixed v0.5.2; a plugin's first `just release` bumps one past its scaffolded version — open, unfixed"
metadata: 
  node_type: memory
  type: project
  originSessionId: 194d9f85-63c2-4d23-8e81-c87672ce360c
  modified: 2026-08-11T19:11:52.720Z
---
`claude-plugin-dev` (https://github.com/ddaanet/claude-plugin-dev,
latest tag `v0.2.0` shipped 2026-04-29) is the canonical release
infrastructure for the user's Claude Code plugins (`handoff`,
`gitmoji`, eventually others). Repo layout: `release.just` (consumer
release recipe), `version-guard.sh` (PreToolUse hook), `check-version.sh`
(plugin.json vs marketplace.json drift check, `just check-version` +
auto pre-flight in `release`), `install.sh` (one-shot vendor + wire),
`tests/hook-test.sh` (version-guard + check-version scenarios),
`justfile` (toolkit's own `precommit` AND self-`release` recipe),
`VERSION` (plain text, last-released version), `docs/design.md`, `CLAUDE.md`.
`check-version.sh` was absorbed 2026-07-27 from a gitlore-local script
(generalized: plugin name from `plugin.json`, marketplace path from
`$MARKETPLACE_DIR`) — unreleased at this writing, so no consumer has
adopted it yet. gitlore's own `scripts/check-version.sh` (hardcoded,
buggy) still needs deleting once gitlore runs `just update-plugin-dev`
to a tag that includes it.

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
- **A consumer that also mounts a gitlore `memory` submodule at top
  level collides with `update-plugin-dev` before v0.5.2.** This repo
  itself mounts gitlore memory at path `memory`; `git subtree pull`
  fetches this repo's raw, unprefixed history first, and git's
  on-demand `fetch.recurseSubmodules` then tries to resolve that
  gitlink using the *consumer's* registered `memory` submodule URL
  instead of this repo's — `fatal: … not our ref …`, aborting the pull
  cleanly before any merge. Fixed in v0.5.2 (`release.just` now runs
  `git -c fetch.recurseSubmodules=no subtree pull …`), but pulling
  v0.5.2 itself from an older vendored copy still needs the same
  override done manually first: `git config fetch.recurseSubmodules
  no` before `just update-plugin-dev`, `git config --unset
  fetch.recurseSubmodules` after — the fix only takes effect once the
  *new* `release.just` is the one running.
- **A plugin's first `just release` bumps one past its scaffolded
  version — open, unfixed.** `release.sh` always bumps forward from
  whatever version is in `plugin.json`, treating it as the *last
  released* version. But a freshly scaffolded plugin (e.g. via the
  unrelated official `plugin-dev:create-plugin` skill) starts at
  `0.1.0`, so a first `just release patch` lands at `0.1.1`, not the
  conventional first-release `0.1.0`. Worked around once (`prohibitions`
  0.1.0, 2026-08-11) by hand-creating tag `v0.1.0` on HEAD then running
  `just resume-release`, which skips the bump/commit step and only
  needs the target tag to already exist locally. Brief filed at
  `claude-plugin-dev/plans/brief-first-release-version-conflict.md`
  proposing a real fix; not yet landed.
- **Toolkit self-release** (since v0.2.0): `just release` in the
  toolkit's own `justfile` bumps `VERSION`, commits, tags, pushes,
  and creates a GitHub release. No more manual `git tag` dance.
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
