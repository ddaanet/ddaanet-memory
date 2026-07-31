---
name: slice-uncommitted-tree-via-index
description: "To slice an uncommitted working tree into commits, stage each slice into the index and never touch the tree — `git checkout --` is blocked as destructive and would discard the only copy"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 57e023c7-06cb-48c0-afd2-2d4f1906467a
  modified: 2026-07-22T21:07:44.712Z
---

Slicing a big **uncommitted** working tree into commits is not the same
problem as re-slicing landed history ([[slice-history-forward-not-by-cherry-pick]]).
There the final state is safe in git; here it exists in exactly one
place, so the "reset to base and rebuild forward" move is unavailable —
`git checkout -- <dir>` is blocked by the permission classifier as
destructive, and rightly so.

Work **index-only** instead. The working tree stays at the final state
from first commit to last:

- Whole-file slices: `git add <paths>`, commit, repeat.
- A file spanning two slices: write the intermediate content to the
  scratchpad, then
  `blob=$(git hash-object -w <file>)` and
  `git update-index --cacheinfo 100644,$blob,<repo-relative-path>`.
  That stages exact bytes without a hand-counted `git apply` hunk split,
  which is where this otherwise goes wrong.
- Last slice: `git add -A <dir>` sweeps the remainder.

**Why:** the post-B2 ghmem wave — 706 insertions across 25 files, six
slices. Only two files needed splitting, and both were one Edit on a
scratchpad copy rather than a patch surgery.

**How to apply:** tar the tree to the scratchpad first and `diff -r` the
extract against it, so the backup is *proven* before any staging starts.
Afterwards, the same `diff -r` against HEAD is the correctness claim —
the slicing changed how the work is told, not what it is.

Intermediate commits cannot be gate-tested in place, since the tree is
never at an intermediate state. Verify after the fact with
`git worktree add --detach <sha>`, running the main tree's venv against
it: `(cd <wt>/tools/ghmem && PYTHONPATH=<wt>/tools/ghmem/src
<main>/.venv/bin/python -m pytest)`. PYTHONPATH wins over the editable
install, so the worktree's own sources are what get imported. Related:
[[user-owns-branching]], [[never-no-verify]].
