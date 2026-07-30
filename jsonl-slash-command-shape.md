---
name: jsonl-slash-command-shape
description: "slash commands log as `<command-name>/foo:bar</command-name>` in a `type:\"user\"` entry, not literal text, no `Skill` tool_use; agent-invoked skills ARE; skill bodies inject as `isMeta:true` either way; `/compact` double-records"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6afc4947-de16-4c6c-9503-de03977fa611
  modified: 2026-07-21T06:57:46.777Z
---

In Claude Code session JSONL, a slash command like `/handoff:handoff`
is recorded as a `type:"user"` entry whose `message.content` is a
**string** containing wrapper tags, e.g.
`<command-name>/handoff:handoff</command-name>\n<command-message>handoff:handoff</command-message>\n<command-args>…</command-args>`
(leading slash present in `<command-name>`). It is NOT stored as the
literal text `/handoff:handoff`, and the slash path does NOT emit a
`Skill` tool_use.

By contrast, agent-invoked skills appear as an assistant `tool_use`
block: `{"type":"tool_use","name":"Skill","input":{"skill":"handoff:handoff"}}`.

**Exception — built-in local commands double-record.** `/compact` (verified
in 5 invocations across 4 sessions, 2026-07-17..20, manual and
watcher-typed alike) logs BOTH a plain `type:"user"` entry whose content is
the literal command line AND the `<command-name>` wrapper record,
~15–60ms apart under different promptIds — one submission, two records,
both rendered (reads as "posted twice"). Queued input gets its own entries:
`type:"queue-operation"` (`enqueue`/`dequeue`, with content) and
`promptSource:"queued"` on the eventual prompt — the authoritative signal
that a submission landed while the session was busy/finalizing.

**Skill bodies are injected as `isMeta:true` user entries** — on BOTH
paths: the `Skill` tool (the body entry also carries `sourceToolUseID`)
and the `/slash-command` path (no `sourceToolUseID`). The body is a
`content:[{type:text,…}]` block. Crucially it starts with the skill's
OWN first line (e.g. `# Update Config Skill`), not always
`"Base directory for this skill:"` — that preamble is only on some
(superpowers-style) skills, so a text-prefix filter misses native skill
bodies. A native body can be 100+ KB. `isMeta:true` is the robust
structural flag for ALL harness-injected non-conversation content (skill
bodies, `<local-command-caveat>`, `## Context`/`## Context Usage`
injections); genuine user prompts never carry it. (Historically the
removed `extract.py` dropped `isMeta`+`isSidechain` to keep skill content
out of the handoff — verified across 20+ real transcripts 2026-05-23.)

The handoff frame no longer scrapes the transcript for content, so `isMeta`
filtering is no longer used anywhere in the plugin. The one live scraper is
`handoff_activated()` in `scripts/_lib.sh`, an activation *detector* (not a
content extractor): it skips `isSidechain` entries and returns true on
either the agent path (a `Skill` tool_use with `input.skill` in
`handoff`/`handoff:handoff`) or the user path (a `type:"user"` entry whose
string content contains `<command-name>/handoff:handoff</command-name>`). It
does not inspect `isMeta`.

Verified against real transcripts 2026-05-23; `handoff_activated()` state
re-confirmed against live `_lib.sh` 2026-07-17. Relevant whenever detecting
skill activation or command invocation from a transcript. Sibling of
[[jsonl-sidechain-segregation]].
