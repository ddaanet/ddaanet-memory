---
name: reference_gitlore_tier_merge_direction
description: a tier fast-forward leaves the root index stale and the next edit silently reverts it — apply upstream tier lines UP into root MEMORY.md as part of the merge; a root-only line deletion returns from the carrier
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4d44ae29-6174-4981-8edd-c8bc5e4cff28
  modified: 2026-07-28T20:38:25.731Z
---

Root `MEMORY.md` is canonical over a tier's carrier index. Composition mirrors
root-authored tier lines **down**, and pulls up only lines missing from root —
it never updates the text of a line root already has.

**A tier fast-forward is therefore not a merge.** When the shared tier advances
from another repo, the tier files move but root keeps its old wording. Nothing
warns you: the resolver reports `state is healthy`, because tier and origin
agree. The staleness is invisible until the next unrelated edit to root, at
which point composition treats stale root as canonical and rewrites the tier
back — silently reverting every incoming line. Symptom: a one-line edit
produces a diff of dozens of files against `origin/live`, and upstream's
commits are undone without a conflict ever being raised.

**Apply the upstream tier lines up into root as part of the merge.** Rebuild
root's tier block from the tier index (strip the carrier's bare paths, re-add
the `<tier>/` prefix), and keep root's own bare-path project lines. Then the
two agree and composition is a no-op reorder.

**Deleting a line needs both files.** A pointer removed from root alone comes
back on the next compose, because it still exists in the carrier and counts as
a line root is missing. The `Edit` reports success and the line reappears.
Delete from the carrier first, then root.

**Correct content *after* propagating it, never during.** A merge that also
"fixes" wording is indistinguishable from the revert above. Land upstream's
lines first, then author corrections in root as their own change, which then
mirrors down legitimately.

*After* is the second step, not permission to skip it. Adoption is per line:
taking upstream's version of the artifact settles the merge, not the merits of
every line in it. A defect verified in an adopted line is the follow-up change
this prescribes — deferring it to a later pass ships it, and the merge commit
makes it look decided.

Related: [[feedback_examine_evidence_drift]] (get the drift direction from git
history — the longer text is not automatically the newer one),
[[feedback_index_compaction_triggers]] (what a corrective pass must restore),
[[feedback_verify_at_source]].
