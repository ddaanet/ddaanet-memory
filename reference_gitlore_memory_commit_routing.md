---
name: reference_gitlore_memory_commit_routing
description: "gitlore: the commit TRIGGER file is what makes memory commit standalone — writing the approved summary alone lets the next parent commit bundle memory with the source it documents; summary mtime must be newer than every memory file or the parent commit aborts"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 86760b03-2500-4f78-85bd-7e921b6bffaf
  modified: 2026-07-25T14:51:27.929Z
---

gitlore routes a memory commit down one of two paths, and the only thing that
selects between them is whether `.claude/gitlore-commit-memory` (the trigger)
is written:

- **summary + trigger** — `PostToolBatch` (`cc-hooks/memory-commit-batch.sh`)
  runs `commit-memory.sh` immediately. Memory lands as its own commit, before
  any parent commit.
- **summary alone** — nothing happens yet. Memory stays dirty with an approved
  summary, and the parent repo's pre-commit hook
  (`gitlore_sync_memory_to_live`, `scripts/lib/resolve.sh`) commits the
  submodule and stages the moved gitlink **into that same parent commit**.

So bundling memory with the source it documents ([[feedback_bundle_memory_with_source]])
is not a matter of ordering two commits — it is a matter of not writing the
trigger.

Two consequences worth knowing before choosing the second path:

- A dirty memory worktree with **no** fresh approved summary makes the
  pre-commit hook return 1, which **aborts the parent commit**. Deferring to
  the parent commit still requires writing the summary file.
- Freshness is `summary mtime >= newest file under the memory worktree`
  (`gitlore_commit_msg_freshness`, `scripts/lib/util.sh`). Any memory edit made
  after the summary file stales it and aborts the commit, so the summary is
  written last.

Related: [[feedback_gitlore_memory]], [[reference_gitlore_bash_edit_desync]].
