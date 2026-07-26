---
name: feedback-bundle-memory-with-source
description: "commit a memory update in the same parent commit as the source change it documents, not separately"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5c5b043b-8460-4841-a796-694a07a15915
  modified: 2026-07-24T17:16:24.725Z
---

When a session both fixes something in source and writes a memory entry
documenting the fix, commit them together — one parent-repo commit carrying
the source diff and the memory submodule's gitlink bump, not two.

**Why:** after fixing a bug in `handoff`'s compaction watcher, I proposed
committing the source fix and the memory submodule's gitlink bump (from
unrelated earlier work) as two separate commits, offering the split as a
choice. The user rejected the premise: a memory update belongs with the
source change it corresponds to, as a standing rule, not a per-commit
judgment call.

**How to apply:** when memory content and source content are written in the
same session for the same reason (a memory note documents a fix, a decision,
or a discovered mechanism), stage and commit them together. This does not
license bundling *unrelated* memory work (e.g., a gitlink bump from an
earlier, separate task) into a commit for a later, separate fix — bundle
what corresponds, not everything pending.
