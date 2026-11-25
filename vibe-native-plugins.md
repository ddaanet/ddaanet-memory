---
name: vibe-native-plugins
description: "porting a Claude Code plugin to Mistral Vibe: a root `plugin.json` with the Agent Plugins 1.0 `$schema` loads natively and wins over a `.claude-plugin/` marker in the same tree, extra markers load as foreign (skills + MCP servers only); plugin hooks live in `ai.mistral.vibe/hooks.toml` and run in the plugin root with `$PLUGIN_ROOT` exported, learning the session cwd only from stdin JSON; plugin `bin/` is never on PATH — the skill load's 'Base directory' line is the entry-point anchor; `.git` is ignored in the plugin digest, so a release-tag clone installs as-is"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8d5debc5-274f-1d8a-16d1-611896bee9f8
  modified: 2026-11-25T11:50:07.000Z
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

## Hooks: no SessionStart-class event, plugin-root cwd

Native hook events are `post_agent`, `pre_tool`, `post_tool` only. A plugin's
`ai.mistral.vibe/hooks.toml` hooks run **in the plugin root** with
`PLUGIN_ROOT`/`PLUGIN_DATA` exported — the session cwd arrives only in the
stdin JSON's `cwd` field, so a CC hook port that leans on
`CLAUDE_PROJECT_DIR` must instead read cwd from stdin. `pre_tool` can rewrite
`tool_input` (composes across hooks); `post_tool` can append
`additional_context`; subagents route through `task`, so `pre_tool match
"task"` reaches their prompts.

## Skills: no plugin bin on PATH

A plugin's `bin/` directory exposes nothing in Vibe (no PATH injection, no
tool surface). The anchor is the base directory text Vibe prints when a skill
loads — skill bodies that must run plugin scripts invoke
`<base>/../../bin/<name>`. `.git` is in the universal runtime-state ignore
set, so a plugin directory that is a `--branch dist-vX.Y.Z` clone digests as
its files alone: the clone itself is the snapshot install.
