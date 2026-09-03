---
name: gitlore-memory-administration-no-parent-commit
description: "administering memory alone (merge/trim/curation, no parent-repo content change): write the approved `.claude/gitlore-memory-message` and stop, don't manufacture a parent commit to force it through"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ae08eab3-deb3-493d-8835-4c8c741651c2
  modified: 2026-09-02T22:12:48.963Z
---

Approving a memory-commit summary (writing it to `.claude/gitlore-memory-message`) is enough on its own to get memory committed. A parent-repo commit is a separate concern and is not required just to administer memory that has no accompanying parent-repo content change.

**Why:** A parent commit (`git add memory && git commit`) was attempted purely to "unblock" a pending memory-tier merge, after an earlier attempt reported "memory is dirty and has no approved commit summary." Writing the approved summary was what actually mattered — the memory commit landed on its own, without the parent commit ever succeeding. My human partner: "you don't need to commit the parent to commit the memory for administration."

**How to apply:** when memory alone needs committing, write the approved summary and stop — don't manufacture an unrelated parent commit to force it through. Reach for a parent commit only when the parent tree itself has content that must land: a real source change, or a gitlink-cleanliness requirement some other tool enforces (e.g. `git subtree pull`'s dirty-tree check, which the resting `M memory` pointer alone can fail).
