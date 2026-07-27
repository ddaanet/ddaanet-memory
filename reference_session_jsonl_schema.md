---
name: session-jsonl-schema-reference
description: "no official schema; key on `isMeta`/`isSidechain`/`isCompactSummary`, never content; one assistant message = N entries sharing `message.id`, so group on it before asking what was batched"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 418b085a-9253-496e-8e70-8f9e0500107c
  modified: 2026-07-26T10:23:40.998Z
---

There is NO authoritative schema for Claude Code session JSONLs
(`~/.claude/projects/<project>/<session>.jsonl`): the official sessions doc
(code.claude.com/docs/en/sessions.md) calls the entry format internal and
version-unstable. Best maintained references (verified 2026-07-12):

- https://claude-dev.tools/docs/jsonl-format — field-level community doc
  (types, uuid/parentUuid, message.content shapes, usage).
- https://github.com/simonw/claude-code-transcripts — actively maintained
  parser (v0.6, Jan 2026); the *code* is the reference, the README only
  documents usage.

For extraction work: verify shapes against real local transcripts and key
filtering on structural flags (`isMeta`, `isSidechain`, `isCompactSummary`),
never content heuristics.

**One assistant message spans several JSONL entries, sharing `message.id`.** A
turn issuing N parallel tool calls is written as N `type:"assistant"` entries,
each carrying one `tool_use` block. So any question about what the model did *in
one message* — did it batch these two writes? how many tools in this call? —
must group on `message.id` first. Counting per entry makes every message look
like a single tool call and silently reports 0% batching; measured 2026-07-26
against 2400 transcripts, per-entry counting said "never batched" where
grouping said 43%.

Compaction appends in the same file/session ID:
`type:"system", subtype:"compact_boundary"` + summary as a `type:"user"`
entry with `isCompactSummary: true` (no `isMeta`). See
[[jsonl-slash-command-shape]] and [[jsonl-sidechain-segregation]].
