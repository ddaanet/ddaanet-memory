---
name: bundle-memory-with-source
description: "which commit a memory update rides — not unrelated pending work"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5c5b043b-8460-4841-a796-694a07a15915
  modified: 2026-08-11T18:10:43.441Z
---

When a session both fixes something in source and writes a memory entry
documenting the fix, commit them together — one parent-repo commit carrying
the source diff and the memory submodule's gitlink bump, not two.

**Why:** after a bug fix in `handoff`'s compaction watcher, the source fix
and the memory submodule's gitlink bump (from unrelated earlier work) were
proposed as two separate commits, with the split offered as a choice. My
human partner rejected the premise: a memory update belongs with the source
change it corresponds to, as a standing rule, not a per-commit judgment
call.

**How to apply:** when memory content and source content are written in the
same session for the same reason (a memory note documents a fix, a decision,
or a discovered mechanism), stage and commit them together. This does not
license bundling *unrelated* memory work (e.g., a gitlink bump from an
earlier, separate task) into a commit for a later, separate fix — bundle
what corresponds, not everything pending.
