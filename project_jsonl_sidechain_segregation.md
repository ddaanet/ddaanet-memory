---
name: jsonl-sidechain-segregation
description: Claude Code main-session JSONLs do not interleave sidechain entries; sub-agent rollups live in separate per-agent files
metadata: 
  node_type: memory
  type: project
  originSessionId: 7bed82ba-de53-4fd5-9e88-49823dea7260
---

As of 2026-05-17, scanning all `~/.claude/projects/*/[0-9a-f-]*.jsonl`
files: no main session JSONL contains `"isSidechain":true`. The
field exists on every entry but is uniformly `false`. Sub-agent
transcripts live in `archived/agent-<hash>.jsonl` files where
**every** entry is sidechain.

**Why:** Drives the extract.py design — filtering `isSidechain` at
load is defence-in-depth, not a fix for current breakage. The
docstring/comment in `load_entries` reflects this.

**How to apply:** When working on handoff extraction or anything
else that parses session JSONLs, treat sidechain segregation as the
*current* invariant but not a permanent one. If a future Claude Code
release interleaves sub-agent activity into the main session JSONL,
the filter at `load_entries` already covers it. Re-verify the
invariant before relying on it for new logic.
