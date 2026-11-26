---
name: vibe-native-plugins
description: "porting a Claude Code plugin to Mistral Vibe: a root `plugin.json` with the Agent Plugins 1.0 `$schema` loads natively and wins over a `.claude-plugin/` marker in the same tree, extra markers load as foreign (skills + MCP servers only); plugin `ai.mistral.vibe/hooks.toml` hooks are parsed into the snapshot but never executed in 3.0.1 (Core gets no hook bindings — no execution adapter yet), so hooks must ship as the project `.vibe/hooks.toml` foreign hooks, which do fire in the session cwd under the command sandbox; plugin `bin/` is never on PATH — the skill load's 'Base directory' line is the entry-point anchor; `.git` is ignored in the plugin digest, so a release-tag clone installs as-is"
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

## Hooks: plugin hooks.toml is parsed but never executed

Native hook events are `post_agent`, `pre_tool`, `post_tool`. The plugin-loader
side exists: `_load_runtime_hooks` (`vibe/core/plugins/_native.py`) parses
`ai.mistral.vibe/hooks.toml`, caps it at 128 hooks / 64 KiB, and prefixes each
name with the plugin name. But the harness never runs any of it:
`core_plugins` (`vibe/app_server/_plugins.py`, 3.0.1) withholds tool groups
and hook bindings from Core — "Core calls what it advertises, and neither has
an execution adapter yet". Probed live 2026-11-25 (Vibe 3.0.1, headless
`vibe -p`): the loaded plugin contributed all six skills and zero hook
bindings. **A plugin cannot carry hooks; hooks ship as foreign `hooks.toml`:**

- `<repo>/.vibe/hooks.toml` (trusted folder) fires; `~/.vibe/hooks.toml` fires
  too, and a same-named user entry wins over the project one (probed
  2026-11-25, the name in both scopes: only the user hook ran) — a user-side
  shadow removes a hook from every repo at once, no per-repo opt-out. Run in
  the **session cwd** under the **command sandbox**, failing open: a hook
  writing outside the workdir failed silently (stderr unexamined), an
  in-workdir write succeeded, and a command path that does not exist leaves
  the session clean — so hook state files belong inside the
  boundary, and a hook script must be found by absolute path, not
  `$PLUGIN_ROOT` (that env is only injected for plugin components, which
  cannot run).
- stdin JSON carries `cwd`, `session_id`, `parent_session_id`, `tool_name`,
  `tool_input`, `tool_status` — the session cwd is on stdin, and a pre_tool
  hook may return `{"decision":"allow","hook_specific_output":{"tool_input":
  {...}}}` for a full **rewrite** (verified end-to-end: a re-written
  `read_file` `path` is what the tool read); `post_tool` may append
  `hook_specific_output.additional_context`.

## Skills: no plugin bin on PATH

A plugin's `bin/` directory exposes nothing in Vibe (no PATH injection, no
tool surface). The anchor is the base directory text Vibe prints when a skill
loads — skill bodies that must run plugin scripts invoke
`<base>/../../bin/<name>`. `.git` is in the universal runtime-state ignore
set, so a plugin directory that is a `--branch dist-vX.Y.Z` clone digests as
its files alone: the clone itself is the snapshot install.
