---
name: reference_gitlore_memory_commit_routing
description: "the TRIGGER file makes a memory commit standalone; withholding it defers to the next parent commit, saving a call but NOT fixing history (the parent stages the gitlink either way); dirty memory with no fresh summary ABORTS that commit, and summary mtime must beat every memory file; neither path pushes — the store's push rides the parent PUSH, so check `origin/live`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 86760b03-2500-4f78-85bd-7e921b6bffaf
  modified: 2026-07-27T22:19:56.187Z
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

Both paths bundle in the parent. The pre-commit hook stages the moved gitlink
unconditionally (`scripts/git-hooks/pre-commit`), so a memory commit made
earlier by the trigger is still recorded by the next parent commit alongside its
source change — one parent commit either way, and
[[feedback_bundle_memory_with_source]] is satisfied by both. Withholding the
trigger is an economy, not a correctness fix: one call instead of two.

Neither path pushes. `commit-memory.sh` commits the submodule and advances
**local** `live` only; the store's push rides the parent repo's `git push`, not
the parent commit. So after a trigger-driven commit plus a parent commit, the
store still reads `HEAD == live != origin/live` — the memory commit exists
nowhere but the local clone until the parent is pushed. Check `origin/live`, not
`live`, before calling memory durable.

Each level is a separate repo needing its own commit, and a nested tier makes
three: the tier store commits and pushes on its own (a merge resolution pushes
it directly), the enclosing `gitlore-memory` store then needs a commit to record
the moved tier gitlink plus its recomposed root index, and the parent repo needs
one to record the store. A gitlink bump with no source change to bundle with is
legitimate at each step — [[feedback_bundle_memory_with_source]] asks for
bundling where a source change exists, and a merge has none.

Two consequences worth knowing before choosing the second path:

- A dirty memory worktree with **no** fresh approved summary makes the
  pre-commit hook return 1, which **aborts the parent commit**. Deferring to
  the parent commit still requires writing the summary file.
- Freshness is `summary mtime >= newest file under the memory worktree`
  (`gitlore_commit_msg_freshness`, `scripts/lib/util.sh`). Any memory edit made
  after the summary file stales it and aborts the commit, so the summary is
  written last.

Related: [[feedback_gitlore_memory]], [[reference_gitlore_tier_merge_direction]].
