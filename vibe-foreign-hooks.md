---
name: vibe-foreign-hooks
description: "Vibe hooks: plugin `ai.mistral.vibe/hooks.toml` is never executed (3.0.1); project/user `.vibe/hooks.toml` scope, cwd, sandbox, authorable points, `vibe -c` reload"
metadata:
  node_type: memory
  type: reference
  originSessionId: f16a6bed-a953-4e46-94d8-6ea12bb015d3
  modified: 2026-11-28T14:26:47.145Z
---

Pinned 2026-11-25 against the installed mistral-vibe source
(`vibe/core/plugins/_compatibility.py`, `_native.py`, the
`mistralai_vibe_local_harness` hooks runner) and a
`claude plugin validate --strict` probe.

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
  too; a hook named in both fires once, from the project entry in a trusted
  folder (the user one is dropped as `Duplicate hook name`, logged only in the
  session export; `vibe/core/hooks/config.py` loads project-first, probed
  2026-11-27), from the user entry in an untrusted one, and differently named
  hooks in both scopes all fire. Run in
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
- `vibe -c` resumes with the same session id **and reloads
  `.vibe/hooks.toml` at resume** — a hook whose command changed between the
  runs fires in its new form (probed 2026-11-26), so hook wiring is re-read
  per episode, not pinned at the session's start.
- The authorable foreign hook points compile to exactly three (`_hook_config.py`
  `_AUTHORABLE_POINTS`): `pre_tool`, `post_tool`, `post_agent` — no session-start
  or `pre_agent_turn` foreign point exists, so nothing runs before the
  session's first tool call, and `match` is one anchored fnmatch glob or a
  `re:` fullmatch regex (one entry can select `re:^(edit|write_file|bash)$`).
