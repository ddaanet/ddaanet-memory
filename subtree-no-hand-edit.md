---
name: subtree-no-hand-edit
description: Never hand-edit vendored plugin-dev/ subtree copies; propagate changes via a tagged toolkit release + update-plugin-dev
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 32f44f4e-e634-4e25-b276-e948b95f86ef
  modified: 2026-07-31T18:52:08.844Z
---

Consumer plugins vendor claude-plugin-dev via `git subtree` under
`<consumer>/plugin-dev/`. Those files are read-only generated content. Every
change goes through the source: edit it in claude-plugin-dev, cut a tagged
release (`just release [patch|minor|major]`), then run `just update-plugin-dev
vX.Y.Z` in each consumer, which wraps `git subtree pull --squash`.

**Why:** a direct edit under `<consumer>/plugin-dev/` recreates precisely the
drift the subtree model prevents — the copy diverges from the tagged source
and from every other consumer, the fix reaches nobody else, and the next
subtree pull conflicts. The single source of truth is
`ddaanet/claude-plugin-dev` at a tag, nowhere else.

**How to apply:** treat a `plugin-dev/` path as unwritable, including for a
one-line fix that "will be released later anyway". Always pin a tag, never
`main`, so a consumer's old tags still resolve to the content vendored at the
time.
