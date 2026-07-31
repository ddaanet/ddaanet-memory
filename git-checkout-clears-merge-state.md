---
name: git-checkout-clears-merge-state
description: "`git checkout` silently unlinks MERGE_HEAD/MERGE_MSG (`remove_branch_state`), so a *clean* `merge --no-commit` dies on any later re-checkout while its index survives; recover by restoring both pointers after `write-tree` == `merge-tree --write-tree`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: dba9cb7c-f425-48f2-aed8-62c27c56bdd5
  modified: 2026-07-26T10:01:07.321Z
---

`git checkout` calls `remove_branch_state()`, which unlinks `MERGE_HEAD`,
`MERGE_MSG`, `CHERRY_PICK_HEAD`, `REVERT_HEAD` and `BISECT_LOG`. It does this
**silently and on success**, with no warning that a merge was in progress.

The dangerous case is a `git merge --no-commit` that auto-merged **cleanly**.
With no unmerged index entries, checkout has nothing to refuse over, so a
later `checkout <same-commit>` — including a no-op re-checkout of the commit
HEAD is already on — succeeds and wipes the merge pointers. The staged merge
result survives in the index; only the second-parent link and the message are
gone. Tooling that keyed on "a merge is prepared" then reports an
unrecoverable state, because its own discriminator (`rev-parse --verify
MERGE_HEAD`) is what was destroyed.

A conflicted merge is safe: unmerged entries make checkout refuse.

**Recovery.** The index is usually still correct — prove it rather than
assume:

    git write-tree                                   # staged result
    git merge-tree --write-tree <ours> <theirs>      # what git would produce

Identical trees mean nothing was lost. Then restore the two pointers and
commit normally:

    printf '%s\n' "<theirs-sha>" > "$(git rev-parse --git-dir)/MERGE_HEAD"
    printf "Merge commit '<theirs-sha>' into HEAD\n" > "$(git rev-parse --git-dir)/MERGE_MSG"
    git commit --no-edit

This preserves the two-parent topology. Committing without restoring
`MERGE_HEAD` produces a single-parent commit that still fast-forwards, but
orphans the merged-in history — the content lands, the ancestry does not.

**Writing tooling.** Any hook or script that re-checkouts a repository on a
schedule (session start, sync loop, watcher) must guard on a merge in
progress before it does. Prepared-merge state and periodic re-checkout are
mutually destructive by design. See [[git-replay-hooks]] for the
adjacent case: which replay operations fire hooks and how to detect them.
