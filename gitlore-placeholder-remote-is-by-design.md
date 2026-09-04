---
name: gitlore-placeholder-remote-is-by-design
description: "a store with no `origin` on a `./.git/gitlore-placeholder` url is a finished install, not a stalled migration: the parent has no remote, so read the PARENT's `git remote -v` before filing it as pending work"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f0bc67da-0b68-4397-87e5-7362cdbe13f2
  modified: 2026-09-02T22:12:44.492Z
---

gitlore's install skips memory-remote creation when the parent repo has no
`origin`. The store then stays local-only and `.gitmodules` keeps the
`./.git/gitlore-placeholder` url it was seeded with. Nothing is unfinished:
commits, the approval gate and tier merges all work against local refs, and
only publishing has nowhere to go.

**The tell is on the parent, not the store.** An empty `git remote -v` in the
parent means local-only by design. Surveying stores alone cannot separate that
from an interrupted migration, which is why a store-side sweep keeps
manufacturing the same false task.

**How to apply:** the pairing worth raising is the reverse one — a parent that
has a remote whose store does not. It does not hold symmetrically: a store can
be given a remote by hand while its parent still has none, so a parent's
missing remote is not a prediction about the store's url. Read both before
concluding either way. Related: [[gitlore-tier-merge-direction]].
