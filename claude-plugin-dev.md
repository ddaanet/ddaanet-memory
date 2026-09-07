---
name: claude-plugin-dev
description: "shared release infra vendored by `git subtree` at a `dist-vX.Y.Z` tag into `plugin-dev/`; the shipped manual `plugin-dev/README.md` owns install, update, dist-tag and gate rules — `unknown dependency prerelease`, the post-pull `just --list`, `plugin-dev/migrations/` — and `plugin-craft:toolkit-release` owns the failure modes; the changelog does not ship"
metadata:
  node_type: memory
  type: project
  originSessionId: 65f754d8-cdbe-42af-8461-7c22a89d764a
  modified: 2026-09-07T13:41:34.361Z
---

`claude-plugin-dev` (https://github.com/ddaanet/claude-plugin-dev) is the
canonical release infra for my human partner's Claude Code plugins
(`handoff`, `gitmoji`, and others), vendored per-consumer via `git subtree`
from a `toolkit/` prefix in the source repo into `plugin-dev/` in the plugin.
`edify` vendors nothing and releases its own way: [[edify-release-flow]].

**Why:** several plugins had diverged near-identical release recipes, and an
agent-driven manual edit to `plugin.json` desynced the manifest from the latest
tag — caught only at release time.

**How to apply:** two documents cover using it, and neither is repeated here.

- **The manual ships with the toolkit.** In a consumer it is
  `plugin-dev/README.md`, in the source repo `toolkit/README.md`, and it
  already answers installing, updating, the two-tag `vX.Y.Z` / `dist-vX.Y.Z`
  scheme and why only a `dist-` ref is vendorable, the `precommit`/`prerelease`
  contract (`unknown dependency prerelease`), migration notes under
  `plugin-dev/migrations/`, and the post-pull `just --list` check. Read it
  rather than reconstructing any of it from memory. Before a first install
  there is no vendored copy — the source repo's root `README.md` carries the
  same install block, and so does the raw file the install command fetches.
- **What goes wrong while running it is the `plugin-craft:toolkit-release`
  skill** — the sandbox and classifier failures that leave a release
  half-landed rather than failed, and what `error: uncommitted changes`
  actually excludes.
- **The changelog does not ship.** A pull crossing more than a patch is worth
  reading `docs/changelog.md` for: its pointer lines say outright when a release
  is breaking, which the optional per-release migration note may not. It exists
  only in the source repo —
  `git -C ~/code/claude-plugin-dev show <tag>:docs/changelog.md` — never under
  `plugin-dev/`, whose tree is the dist split and carries no `docs/`.
