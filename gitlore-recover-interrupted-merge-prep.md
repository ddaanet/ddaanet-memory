---
name: gitlore-recover-interrupted-merge-prep
description: "gitlore push-memory.sh killed/timed-out mid-run can orphan a store's MERGE_HEAD with no gitlore gate able to see it; recovery reconstructs gitlore's own merge-state JSON and re-enters its real continue-after-merge, rather than hand-rolling the commit/compose/publish"
metadata:
  node_type: memory
  type: reference
  originSessionId: 698f43da-45cf-452a-ac3a-b5abc55a1bc6
  modified: 2026-08-12T11:29:51.174Z
---

`push-memory.sh`/`resolve.sh`'s `gitlore_prepare_merge` reads the pending
commit from `HEAD`, not the store's `live` branch. An interruption after its
`checkout --detach <authority>` but before `gitlore_write_merge_state` writes
the JSON state file leaves a real, clean `MERGE_HEAD` that
`gitlore_guard_stale_merge_state` cannot detect (it only checks the state
file's presence). The next run's own checkout silently wipes that `MERGE_HEAD`
(git's `remove_branch_state` — see [[reference_git_checkout_clears_merge_state]])
and misreads `pending` as the already-moved `HEAD`, producing "Already up to
date." / "could not prepare the memory merge" on every retry. Full analysis
and repro: `gitlore` repo,
`brief-orphaned-merge-head-no-state-file.md`. As of `afb02b9` gitlore's push
path gates on `HEAD`-vs-`live` agreement before publishing, which catches
most shapes of this early — but not all, and the commit-time sync path
(`gitlore_sync_tiers_to_live`/`gitlore_sync_memory_to_live`) has no such gate
at all.

**Recovery**, once `git write-tree` in the store confirms the staged
merge result matches a fresh `git merge-tree --write-tree <authority>
<pending-from-live-branch>` (nothing lost): restore `MERGE_HEAD`/`MERGE_MSG`
by hand, then reconstruct gitlore's own merge-state JSON — source
`scripts/lib/{util,log,resolve}.sh` from the installed plugin and call
`gitlore_write_merge_state "$store" "<flavor>" "$base" "$pending" "$authority"
"continue-after-merge"` directly — and run the real `bash
$PLUGIN_ROOT/scripts/resolve.sh continue-after-merge` from the parent repo
root. Do not hand-roll the commit/compose/publish sequence: `continue-after-
merge` also composes the merged tier index up into root `MEMORY.md`
(`compose_merged_indexes`), which a manual `git commit` would skip.
