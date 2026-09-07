---
name: claude-plugin-dev
description: "shared release infra vendored by `git subtree` into `plugin-dev/`; vendor the `dist-vX.Y.Z` tag, never the source `vX.Y.Z`; `install.sh` does the whole first install; `just update-plugin-dev` then `just --list`, because `unknown dependency prerelease` makes just refuse EVERY recipe; migration notes at `plugin-dev/migrations/`"
metadata: 
  node_type: memory
  type: project
  originSessionId: 65f754d8-cdbe-42af-8461-7c22a89d764a
  modified: 2026-09-03T08:48:14.279Z
---

`claude-plugin-dev` (https://github.com/ddaanet/claude-plugin-dev) is the
canonical release infra for my human partner's Claude Code plugins
(`handoff`, `gitmoji`, and others), vendored per-consumer via `git subtree`
from a `toolkit/` prefix in the source repo into `plugin-dev/` in the plugin.
`toolkit/README.md` ships with it and says what each file is for. `edify`
vendors nothing and releases its own way: [[edify-release-flow]].

**Why:** several plugins had diverged near-identical release recipes, and an
agent-driven manual edit to `plugin.json` desynced the manifest from the latest
tag — caught only at release time.

**How to apply:**

- **Two tags per toolkit release**: `vX.Y.Z` is the source tag; `dist-vX.Y.Z`
  is a `git subtree split --prefix=toolkit` of the same commit — root tree is
  consumer-facing files only. Vendoring the source tag copies the toolkit's own
  `memory` submodule, `.claude/`, `CLAUDE.md` and justfile into the plugin.
  **Always vendor/update the `dist-` tag**; `install.sh` and
  `update-plugin-dev` both refuse a `vX.Y.Z` ref and name the `dist-` one to
  use instead. Resolve the newest via
  `git ls-remote --tags --refs --sort=-v:refname <url> 'dist-v*' | head -1`.
- `install.sh` is the whole first install and vendors the toolkit itself. Both
  READMEs carry the block: resolve the newest `dist-` tag, `curl` that tag's
  root `install.sh`, pipe it to `bash -s -- "$tag"` from the plugin root. It
  does the `git subtree add --prefix=plugin-dev … --squash`, wires
  `import 'plugin-dev/release.just'` into the `justfile` and the version-guard
  hook into `.claude/settings.json`. Don't hand-run the `subtree add` first.
- Consumer must define a `precommit` recipe (the commit gate) and a
  `prerelease` recipe (the gate `release` depends on — often just
  `prerelease: precommit`). `just` fails every recipe immediately with
  `unknown dependency prerelease` if the consumer omits it.
- Update with `just update-plugin-dev`, which resolves the newest `dist-` tag
  off the remote; pass one to pin an older version. It rejects a dirty tree and
  accepts *only* a `dist-v*` ref — a source tag is refused naming the `dist-`
  one instead, and a branch, a sha or `main` is refused outright.
  A plugin vendored before dist refs existed carries the toolkit's leaked
  environment under `plugin-dev/` (e.g. a `plugin-dev/memory` gitlink breaking
  a bare `git submodule status`); the first `dist-` pull deletes all of it in
  the same commit, no cleanup needed.
- **Run `just --list` before considering a pull done** — a toolkit version
  that requires a consumer-side change (the `prerelease` recipe was one)
  makes `just` refuse to compile *any* recipe, so the breakage is invisible
  until the next unrelated recipe run. Land the consumer-side fix as its own
  commit, separate from the subtree-pull merge commit.
- A release that needs a consumer-side step ships a migration note at
  `plugin-dev/migrations/vX.Y.Z.md`, and the pull prints every note in the
  version range it crossed. Those are guidance to apply by hand — nothing
  enforces them, which is why the `just --list` check stands regardless. A
  note is optional per release, so crossing more than a patch still rewards
  the toolkit's `docs/changelog.md`, whose pointer lines say outright when a
  release is breaking. Read it from the source checkout
  (`git -C ~/code/claude-plugin-dev show <tag>:docs/changelog.md`), never from
  `plugin-dev/`: the `dist-` tree ships no `docs/`.
