---
name: claude-plugin-dev
description: "installing or updating the claude-plugin-dev toolkit vendored into `plugin-dev/` at a `dist-vX.Y.Z` tag: `plugin-dev/README.md` is the manual; ``Recipe `release` has unknown dependency `prerelease` `` from `just` means the justfile lacks the `prerelease` recipe it documents"
metadata:
  node_type: memory
  type: project
  originSessionId: 65f754d8-cdbe-42af-8461-7c22a89d764a
  modified: 2026-11-17T23:28:38.000Z
---

The `claude-plugin-dev` release toolkit is vendored into each consumer plugin
by `git subtree` at a `dist-vX.Y.Z` tag, into `plugin-dev/`. Its shipped
manual is `plugin-dev/README.md`: read it for installing, updating and the
gate contract rather than reconstructing them. Before a first install there is
no vendored copy; the install block is in the source repository's root
`README.md`, and the command it gives fetches `install.sh` from the same tag.

`just` failing with ``Recipe `release` has unknown dependency `prerelease` ``
(the recipe named is whichever depends on it; every recipe fails) means the consumer's justfile lacks the `prerelease` recipe the
README documents as mandatory: just rejects a justfile whose dependency names a
missing recipe. Add the recipe as the README shows it.

The other release failures — sandbox and classifier refusals that half-land a
release, what `error: uncommitted changes` excludes — are in the
`plugin-craft:toolkit-release` skill.
