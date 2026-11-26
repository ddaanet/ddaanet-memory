---
name: vibe-native-plugins
description: "porting a Claude Code plugin to Mistral Vibe: a root `plugin.json` with the Agent Plugins 1.0 `$schema` loads natively and wins over a `.claude-plugin/` marker in the same tree; plugin `ai.mistral.vibe/hooks.toml` hooks are parsed but never executed in 3.0.1 — hooks must ship as project `.vibe/hooks.toml` foreign hooks, which fire in the session cwd under the command sandbox (cwd on stdin, `pre_tool` rewrites `tool_input`); plugin `bin/` is never on PATH — the skill load's 'Base directory' line is the entry-point anchor; `.git` is ignored in the plugin digest, so a release-tag clone installs as-is; a subagent dispatch travels as `agents.spawn` (hooks observe it via `tool_input` `{name,message}`, rewrites are inert on it, deny works) and the authorable foreign points are pre_tool/post_tool/post_agent only — no session-start hook; on text delivery `systemMessage` is dropped at both tool points and pre_tool's `additional_context` too (rewrite and deny only), post_tool `additional_context` is the one working text channel, and a resumed `vibe -c` reloads `.vibe/hooks.toml`"
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
  in-workdir write succeeded — and as of 2026-11-26 `/tmp` writes (append,
  `mktemp`) from hook commands also succeeded in headless probes, so the
  2026-11-25 silent failure was the session-log scratchpad area, not `/tmp`:
  workdir and `/tmp` are writable from a hook on the observed evidence (one
  observation per target, no measured rule). A command path that does not
  exist leaves the session clean — so hook state files belong inside the
  boundary, and a hook script must be found by absolute path, not
  `$PLUGIN_ROOT` (that env is only injected for plugin components, which
  cannot run).
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
  bash calls alike.
- `vibe -c` resumes with the same session id **and reloads
  `.vibe/hooks.toml` at resume** — a hook whose command changed between the
  runs fires in its new form (probed 2026-11-26), so hook wiring is re-read
  per episode, not pinned at the session's start.
- The authorable foreign hook points compile to exactly three (`_hook_config.py`
  `_AUTHORABLE_POINTS`): `pre_tool`, `post_tool`, `post_agent` — no session-start
  or `pre_agent_turn` foreign point exists, so nothing runs before the
  session's first tool call, and `match` is one anchored fnmatch glob or a
  `re:` fullmatch regex (one entry can select `re:^(edit|write_file|bash)$`).

## Dispatches: hooks see agents.spawn, rewrites are inert on it

Probed 2026-11-26 (Vibe 3.0.1, headless sessions with catch-all and
spawn-targeted hooks). There is no `task`-named tool call in a Vibe session:
a dispatch travels as a programmatic execution of `tools.agents.spawn`
inside `run_typescript` (session record: `toolName: "agents.spawn"`, input
normalized `{agent, task}`). The authoritative matcher names, from
`mistralai_vibe_local_harness/vibe/_hook_matcher.py` `candidate_names`, are
`agents.spawn`, `agents_spawn`, `spawn` (plus `re:`/fnmatch recipes) —
matching is at call time, back-compat legacy names included.

- Hooks DO observe spawns: `pre_tool` and `post_tool` fire with
  `tool_name = "agents.spawn"`, `tool_input = {name, message}` where
  `message` is the child's admitted first message. A denied call fires no
  `post_tool`.
- **Rewrites are inert on the programmatic path**: a hook's
  `hook_specific_output.tool_input` rewrite of `message` reached Core (the
  rewrite was logged) but the child received the original text — the
  programmatic external execution runs the requested arguments, not the
  rewritten ones. Injection into a dispatch prompt is therefore impossible
  via hooks; the only hook-expressible act on a spawn is **deny**, which
  works: no child session is created and the reason reaches the model as
  the tool's `tool_skipped` error while the session continues.
- The child is its own session (reads the repo `AGENTS.md` itself, carries
  `hook_bindings []`), which is why a rewrite would not matter for the
  AGENTS.md-visible parent in-framework, and why nothing injects into a
  subagent in a repo whose AGENTS.md carries no store line.

## Skills: no plugin bin on PATH

A plugin's `bin/` directory exposes nothing in Vibe (no PATH injection, no
tool surface). The anchor is the base directory text Vibe prints when a skill
loads — skill bodies that must run plugin scripts invoke
`<base>/../../bin/<name>`. `.git` is in the universal runtime-state ignore
set, so a plugin directory that is a `--branch dist-vX.Y.Z` clone digests as
its files alone: the clone itself is the snapshot install.
