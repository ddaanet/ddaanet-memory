---
name: vibe-plugin-detection
description: "Vibe plugin format detection: a root `plugin.json` with the Agent Plugins 1.0 `$schema` wins over `.claude-plugin/`; two foreign markers are AMBIGUOUS"
metadata:
  node_type: memory
  type: reference
  originSessionId: f16a6bed-a953-4e46-94d8-6ea12bb015d3
  modified: 2026-11-28T14:26:46.148Z
---

Pinned 2026-11-25 against the installed mistral-vibe source
(`vibe/core/plugins/_compatibility.py`, `_native.py`, the
`mistralai_vibe_local_harness` hooks runner) and a
`claude plugin validate --strict` probe.

## Detection: native manifest wins outright

`detect_plugin_source_format` reads root `plugin.json` first: a matching
`$schema` returns Agent Plugins 1.0 and foreign markers are never consulted.
So one tree can carry both a root native manifest and `.claude-plugin/` —
Claude Code reads the latter, Vibe the former, and the same `dist` branch and
`dist-vX.Y.Z` tags ship both. `plugin validate --strict` tolerates the extra
root files (verified on gitlore's dogfood tree, 2026-11-25). Mixing two
*foreign* markers is the fatal AMBIGUOUS case.
