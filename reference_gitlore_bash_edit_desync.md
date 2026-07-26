---
name: reference-gitlore-bash-edit-desync
description: "`index-compose.sh`/`index-sync-post.sh` fire only on `Write`|`Edit`, so a `sed`-applied `MEMORY.md` edit desyncs root vs. tier and later refuses with \"duplicate pointer path\""
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5c5b043b-8460-4841-a796-694a07a15915
  modified: 2026-07-24T15:58:13.368Z
---

gitlore's `PostToolBatch` hooks (`index-compose.sh`, `index-sync-post.sh`)
filter `.tool_calls[]` for `tool_name == "Write" or "Edit"` before doing
anything — a change applied via Bash (`sed`, sed-Python, redirection) to
`MEMORY.md` or a tier's own `MEMORY.md` is invisible to them. The edit
lands on disk but composition doesn't run, so the root index and a mounted
tier's own index can silently drift apart.

**Observed 2026-07-24:** removing a line from the root `MEMORY.md` with
`sed -i` (to fix a duplicate pointer) left the same duplicate sitting in
`memory/<tier>/MEMORY.md`, since only the root got touched. The next
Write/Edit-tool touch to the root index then triggered composition, which
refused outright with "duplicate pointer path" — the store had to be fixed
by hand in the tier's own file before composition would run again.

**How to apply:** after any Bash-level edit to a gitlore-managed
`MEMORY.md`, make one trivial Write or Edit-tool touch to the same file to
force composition/sync to run and catch drift immediately, rather than
letting it surface later as a refusal or an invisible duplicate.
