---
name: claude-plugin-dev
description: "shared release infra; vendor the `dist-vX.Y.Z` tag, never source; `just --list` after a subtree pull; `error: uncommitted changes` = real work (gate exempts only `.claude` + the memory mount); unsandboxed `just release`, `resume-release` finishes a half-done one; `edify` ports it locally instead of vendoring, for the PyPI step"
metadata: 
  node_type: memory
  type: project
  originSessionId: 194d9f85-63c2-4d23-8e81-c87672ce360c
  modified: 2026-09-01T18:16:52.594Z
---
`claude-plugin-dev` (https://github.com/ddaanet/claude-plugin-dev) is the
canonical release infrastructure for my human partner's Claude Code plugins
(`handoff`, `gitmoji`, and others), vendored per-consumer via `git subtree`
under a `toolkit/` prefix in the source repo. Contents (per `toolkit/README.md`):
`release.just` (thin recipe wrappers a consumer imports: `release`,
`resume-release`, `check-version`, `update-plugin-dev`), `release.sh` (the
release flow — validates state, bumps `.claude-plugin/plugin.json`, commits,
tags, pushes, creates a GitHub release, bumps/creates the marketplace.json
entry; `--resume` probes what already landed and does only what's missing,
a no-op if everything already landed), `check-version.sh` (plugin.json vs
marketplace.json drift check, run as `release`'s pre-flight so it refuses to
start atop an unfinished prior release), `version-guard.sh` (PreToolUse hook
blocking agent edits to `plugin.json`'s `.version`), `install.sh` (one-shot
vendor + wire, idempotent).

**Why:** Several plugins had diverged near-identical release recipes,
and an agent-driven manual edit to `plugin.json` desynced the
manifest from the latest tag — caught only at release time. Toolkit
unifies the recipe, adds a version-guard hook, and folds the marketplace
bump into the same atomic release.

**How to apply:**

- **Two tags per toolkit release**: `vX.Y.Z` is the source tag; `dist-vX.Y.Z`
  is a `git subtree split --prefix=toolkit` of the same commit — root tree
  is consumer-facing files only. Vendoring the source tag would copy the
  toolkit repo's own `memory` submodule, `.claude/`, `CLAUDE.md`, justfile
  into the plugin. **Always vendor/update the `dist-` tag**; `install.sh` and
  `update-plugin-dev` both refuse a `vX.Y.Z` ref and name the `dist-` one to
  use instead. Resolve the newest via
  `git ls-remote --tags --refs --sort=-v:refname <url> 'dist-v*' | head -1`.
- Consumer plugins vendor at `plugin-dev/` via
  `git subtree add --prefix=plugin-dev <url> <dist-tag> --squash`, then
  `bash plugin-dev/install.sh <dist-tag>`, which wires
  `import 'plugin-dev/release.just'` into the consumer's `justfile` and the
  version-guard hook into `.claude/settings.json`.
- Consumer must define a `precommit` recipe (the commit gate) and a
  `prerelease` recipe (the gate `release` depends on — often just
  `prerelease: precommit`). `just` fails every recipe immediately with
  `unknown dependency prerelease` if the consumer omits it.
- Release commit message convention: `release: X.Y.Z` (gitmoji hook
  rewrites to `🔖 release X.Y.Z`).
- Plugin manifest holds the *last released* version; `just release
  {patch|minor|major}` bumps from there. Manual edits are blocked by the
  version-guard hook and the recipe's pre-flight check. A plugin that has
  never been released has nothing to bump from: its first `just release`
  takes no bump argument and publishes whatever version `plugin.json`
  already holds; passing a bump on that first call is refused.
- Update toolkit version in a consumer with
  `just update-plugin-dev dist-vX.Y.Z` (wraps `git subtree pull`; rejects a
  dirty tree, refuses a source-tag ref, warns on a branch ref). A plugin
  vendored before dist refs existed carries the toolkit's leaked working
  environment under `plugin-dev/` (e.g. a `plugin-dev/memory` gitlink
  breaking a bare `git submodule status`); the first `dist-` pull deletes
  all of it in the same commit, no manual cleanup needed.
  **Run `just --list` before considering the pull done** — a toolkit
  version that requires a consumer-side change (the `prerelease` recipe was
  one) makes `just` refuse to compile *any* recipe, so the breakage is
  invisible until the next unrelated recipe run. Land the consumer-side fix
  as its own commit, separate from the subtree-pull merge commit: the merge
  commit is toolkit content, the fix is the consumer repo's own.
  Crossing more than a patch version, read the toolkit's `docs/changelog.md`
  first — its pointer lines carry the version and say outright when a release
  is breaking. Read it from the source checkout at `~/code/claude-plugin-dev`
  (`git -C ~/code/claude-plugin-dev show <tag>:docs/changelog.md` for a
  specific tag), not from `plugin-dev/`: the `dist-` tree ships no `docs/`,
  and the vendored copy stays a version behind until the pull lands.
- Default branch is auto-detected from `origin/HEAD` — recipes don't
  hardcode `main`.
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
  `tree_is_clean` before blaming the tree. This repo's own self-release recipe
  in `justfile` carries the same two exclusions as literals.
- **Run `just release` with the command sandbox disabled.** The
  marketplace bump is the recipe's last step and its only write outside
  the repo, so under the sandbox it dies there on `mv: inter-device move
  failed … unable to remove target: Read-only file system` against the
  marketplace path. Everything before it — version commit, tag, branch
  push, GitHub release — has already landed, so a sandboxed run leaves a
  half-done release rather than a failed one; `just resume-release`,
  likewise unsandboxed, completes it and is idempotent. See
  [[sandbox-effects]]. Unsandboxed is necessary and not sufficient: the
  marketplace lives in a sibling repo, and the push into it is refused by
  the auto-mode classifier as an *external repo outside the trusted source
  control org* regardless of the flag, until my human partner has run
  `/add-dir` on that repo ([[classifier-denied-self-config]]).
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
