---
name: gitlore-tier-merge-direction
description: "tiers are pinned: `/gitlore:merge`|`/gitlore:push` land the merge in the carrier, then adopt it UP into root's block — never rebuild root's block by hand; delete a pointer from root alone; correct wording only AFTER propagating, and a verified defect in an adopted line is the follow-up change"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4d44ae29-6174-4981-8edd-c8bc5e4cff28
  modified: 2026-07-30T15:26:44.458Z
---

A tier is pinned at the commit the memory tree records for it, so it advances
only through `/gitlore:merge` or `/gitlore:push`. Both land the merge in the
tier's carrier index and then adopt it **up** in the same pass: the merged
carrier becomes root's block for that tier. **Adoption is the tooling's** —
don't rebuild root's block from a tier index by hand. What it leaves behind is
a dirty memory store, the moved gitlink and the recomposed root index, and
those ride the next memory commit.

In-session the projection runs the other way, root's lines for an active tier
**down** into the carrier, and a carrier path root's working tree lacks is
decided by root at `HEAD`: present there means root deleted it, absent means
nobody authored it in root, so it is kept and reported. **Removing a pointer
from root alone is therefore enough** — the carrier copy goes in the same pass.

**Correct content *after* propagating it, never during.** A merge that also
"fixes" wording is indistinguishable from reverting what upstream just sent.
Land upstream's lines first, then author corrections in root as their own
change, which then mirrors down legitimately.

*After* is the second step, not permission to skip it. Adoption is per line:
taking upstream's version of the artifact settles the merge, not the merits of
every line in it. A defect verified in an adopted line is the follow-up change
this prescribes — deferring it to a later pass ships it, and the merge commit
makes it look decided.

Related: [[examine-evidence-drift]] (get the drift direction from git
history — the longer text is not automatically the newer one),
[[index-compaction-triggers]] (what a corrective pass must restore),
[[verify-at-source]].
