---
name: reference_gitlore_bash_edit_desync
description: "gitlore's index-compose.sh/index-sync-post.sh only fire on Write|Edit tool_name — a Bash-applied (e.g. sed) MEMORY.md edit desyncs root vs. tier index and later refuses with \"duplicate pointer path\""
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5ca585b4-cdf0-4f60-85ae-1a841693efb4
  modified: 2026-07-28T10:24:56.375Z
---

gitlore keeps a tier's carrier index (`<tier>/MEMORY.md`) and the root
`MEMORY.md` in sync via `PostToolUse` hooks — `index-compose.sh` and
`index-sync-post.sh` — that key off the tool that made the edit, not the file
that changed. They fire only when `tool_name` is `Write` or `Edit`.

**A Bash-applied edit — `sed -i`, a heredoc rewrite, any shell tool touching
`MEMORY.md` directly — never fires them.** The file changes on disk but the
carrier and root indexes fall out of sync silently: no error at edit time,
because nothing was watching that tool.

**Symptom shows up later, not at the desyncing edit.** The next legitimate
`Write`/`Edit` to an index recomposes from both sides and finds the same
pointer already present in one but not reconciled in the other, and gitlore
refuses with `"duplicate pointer path"` — an error that names a line, not the
Bash command that actually caused the drift.

**How to apply:** always edit `MEMORY.md` (root or tier) with the `Write` or
`Edit` tool, never Bash/sed, even for a one-line fix. If a desync has already
happened, don't hand-patch around the refusal — reconcile root and tier by
hand (compare both indexes, decide which side is canonical for the disputed
line) before the next Write/Edit, the same way [[reference_gitlore_tier_merge_direction]]
resolves a stale-root fast-forward.

Related: [[reference_gitlore_tier_merge_direction]] (a different desync
cause — tier fast-forward, not a Bash edit — with the same "recompose finds
disagreement" symptom class), [[feedback_index_compaction_triggers]].
