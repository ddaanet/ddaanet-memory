---
name: session-jsonl-schema-reference
description: "no official schema (docs: internal/unstable); refs claude-dev.tools/docs/jsonl-format + simonw/claude-code-transcripts; key on `isMeta`/`isSidechain`/`isCompactSummary`, never content"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 418b085a-9253-496e-8e70-8f9e0500107c
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
never content heuristics. Compaction appends in the same file/session ID:
`type:"system", subtype:"compact_boundary"` + summary as a `type:"user"`
entry with `isCompactSummary: true` (no `isMeta`). See
[[jsonl-slash-command-shape]] and [[jsonl-sidechain-segregation]].
