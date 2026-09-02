---
name: jsonl-sidechain-segregation
description: "main session JSONLs never carry `\"isSidechain\":true`, so a reader of `transcript_path` needs no sidechain filter; sub-agents live in `<session-id>/subagents/agent-<agent_id>.jsonl`, `archived/` holds retired whole sessions"
metadata:
  node_type: memory
  type: project
  originSessionId: 7bed82ba-de53-4fd5-9e88-49823dea7260
  modified: 2026-08-08T16:34:24.636Z
---

As of CC 2.1.222, a main session JSONL never carries `"isSidechain":true`. The
field is present on every entry and uniformly `false`.

Sub-agent transcripts live at
`~/.claude/projects/<encoded-project>/<session-id>/subagents/agent-<agent_id>.jsonl`,
each beside an `agent-<agent_id>.meta.json`, in a session directory that also
holds `tool-results/`. Every entry in them is `isSidechain:true`. `archived/`
is **not** a sub-agent location: it holds retired whole sessions, each a
`<session-id>.jsonl` plus its own `<session-id>/subagents/` — the same shape,
one level down.

**Why:** a reader handed `transcript_path` sees the main file alone, and a
sub-agent's `message.usage` sits in a sibling directory that nothing appends
from. A `select(.isSidechain != true)` guard on such a reader is code for a
case the layout makes impossible, so it should not be written.

**How to apply:** parse the file at `transcript_path` unfiltered, and treat the
segregation as an invariant of the current layout rather than a permanent one.
Re-verify by disjointness rather than by the flag: collect the `message.id`s
carrying `message.usage` from the main transcript and from its
`subagents/*.jsonl`, and the two sets must not intersect. The flag alone would
still read clean if a later release merged the files under a different marker.
[[session-jsonl-schema]] covers what else is safe to key on.
