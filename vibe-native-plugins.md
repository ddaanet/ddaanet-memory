---
name: vibe-native-plugins
description: "porting a Claude Code plugin to Mistral Vibe: a root `plugin.json` with the Agent Plugins 1.0 `$schema` loads natively and wins over a `.claude-plugin/` marker in the same tree; plugin `ai.mistral.vibe/hooks.toml` hooks are parsed but never executed in 3.0.1 — hooks must ship as project `.vibe/hooks.toml` foreign hooks, which fire in the session cwd under the command sandbox (cwd on stdin, `pre_tool` rewrites `tool_input`); plugin `bin/` is never on PATH — the skill load's 'Base directory' line is the entry-point anchor; `.git` is ignored in the plugin digest, so a release-tag clone installs as-is; a subagent dispatch travels as `agents.spawn` (hooks observe it via `tool_input` `{name,message}`, rewrites are inert on it, deny works) and the authorable foreign points are pre_tool/post_tool/post_agent only — no session-start hook; on text delivery `systemMessage` is dropped at both tool points and pre_tool's `additional_context` too (rewrite and deny only), post_tool `additional_context` is the one working text channel (the snake `system_message` is the user's notice channel — shown, never model-read; camel spellings have no parser reader), and a resumed `vibe -c` reloads `.vibe/hooks.toml`"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8d5debc5-274f-1d8a-16d1-611896bee9f8
  modified: 2026-11-28T14:26:45.524Z
---

Pinned 2026-11-25 against the installed mistral-vibe source
(`vibe/core/plugins/_compatibility.py`, `_native.py`, the
`mistralai_vibe_local_harness` hooks runner) and a
`claude plugin validate --strict` probe.

One fact per file beside this one; read the one matching the question:

- Manifest detection, root `plugin.json` vs `.claude-plugin/`: [vibe-plugin-detection.md](vibe-plugin-detection.md)
- Plugin hooks never run, `.vibe/hooks.toml` scopes, cwd, sandbox, authorable points, `vibe -c` reload: [vibe-foreign-hooks.md](vibe-foreign-hooks.md)
- Hook stdin, `tool_input` rewrite, which output channel reaches the model or the user: [vibe-hook-io-channels.md](vibe-hook-io-channels.md)
- Subagent dispatch as `agents.spawn`, rewrite inert, deny works: [vibe-hooks-agents-spawn.md](vibe-hooks-agents-spawn.md)
- Plugin `bin/` not on PATH, base-directory anchor, `.git` in the digest: [vibe-plugin-bin-path.md](vibe-plugin-bin-path.md)
