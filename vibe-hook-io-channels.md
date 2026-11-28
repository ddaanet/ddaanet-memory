---
name: vibe-hook-io-channels
description: "Vibe foreign hook stdin fields, `tool_input` rewrite, and per-point text delivery: only post_tool `additional_context` reaches the model; snake `system_message` is user-only"
metadata:
  node_type: memory
  type: reference
  originSessionId: f16a6bed-a953-4e46-94d8-6ea12bb015d3
  modified: 2026-11-28T14:26:48.227Z
---

Pinned 2026-11-25 against the installed mistral-vibe source
(`vibe/core/plugins/_compatibility.py`, `_native.py`, the
`mistralai_vibe_local_harness` hooks runner) and a
`claude plugin validate --strict` probe.

## Hooks: stdin, rewrite and delivery channels

- stdin JSON carries `cwd`, `session_id`, `transcript_path`,
  `parent_session_id`, `tool_name`, `tool_input`, `tool_status` — the session
  cwd is on stdin, and a pre_tool hook may return
  `{"decision":"allow","hook_specific_output":{"tool_input":{...}}}` for a
  full **rewrite** (verified end-to-end: a re-written `read_file` `path` is
  what the tool read); `post_tool` may append
  `hook_specific_output.additional_context`, and a post_tool `deny` replaces
  the model-visible result with the reason rather than un-running the call.
- Delivery per point (probed live, Vibe 3.0.1, 2026-11-26): `systemMessage`
  is **dropped at both tool points** — never journaled, never delivered — and
  `pre_tool`'s `additional_context` is dropped too, with or without a
  `"decision":"allow"` envelope, so a pre_tool hook delivers only `tool_input`
  rewrite and `deny`. **post_tool `additional_context` is the one working
  text channel**: it is appended to the tool's result, reaches the model
  verbatim, works at user and project scope, and fires on read, write and
  bash calls alike. The parser's user channel is the snake `system_message` —
  read on both tool points, shown as a hook notice the user sees and the
  model never reads (no camelCase fallback anywhere in `_foreign_hooks.py`);
  the camel spellings (`systemMessage`, `suppressOutput`, `hookSpecificOutput`)
  have no reader at all, so a sysmsg-only report reaches the user alone and a
  camel-emitting hook must be aliased (the D94 `_shim` translation does
  exactly this).
