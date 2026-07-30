---
name: slice-history-forward-not-by-cherry-pick
description: "To re-slice landed straight-line history into review batches, construct each slice's tree forward from the final state; cherry-pick reordering conflicts hopelessly"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: bc8e4078-db3a-4168-8b66-b5c681f7ace1
  modified: 2026-07-21T08:01:01.794Z
---

When a feature landed as many interleaved commits and needs
re-expressing as a few reviewable slices, do NOT cherry-pick the
commits into slice order. It conflicts immediately and everywhere:
later commits rewrite earlier ones wholesale (ghmem's `heat.py` was
created once and then replaced by the aggregate-query rewrite), and
the files every slice shares — models, store, CLI, the ingest reducer —
are touched by every group.

Instead build each slice's tree **forward** from the final state:

1. Branch off the base. `git checkout <final> -- <this slice's files>`.
2. Hand-strip whatever belongs to LATER slices out of the shared files
   (delete the entity classes, the query methods, the CLI command).
3. Run the full gate. Commit. Branch again for the next slice, stacked.

Stacking is what makes the shared files tractable — each slice's diff
is against its parent, so the shared file grows by accretion instead of
needing an independent split per branch.

Two checks that make the result trustworthy:

- **Every slice green on its own**, or the slicing is worthless.
- **The tip's tree byte-identical to the original**: `git diff --stat
  <old-tip> <new-tip>` must be empty. That is the whole correctness
  claim — the rewrite changed how the work is told, not what it is.

Ordering falls out of dependencies, not of history. And **fold bug-fix
commits into the feature they fix**: a reviewer reading a defect
followed by its repair learns nothing the final state does not show,
and the reasoning lives in the docstrings anyway.

**Why:** slicing ghmem's Part A (21 commits, 5 shared files) into 6
commits. The cherry-pick attempt conflicted on the second commit; the
forward construction took one pass per slice with no conflicts at all.

**How to apply:** expect the slice count and boundaries to change once
you build them — a diffstat predicts size, not coherence. Part A's
routing slice looked like one 662-line batch and turned out to be two
distinct arguments (what shape routing state is / how a live write
avoids diverging from a replay) that read better apart.

Related: [[honest-line-count-caps]],
[[spec-contract-size-predicts-pr-size]], [[user-owns-branching]].
