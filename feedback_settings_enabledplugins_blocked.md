---
name: feedback_settings_enabledplugins_blocked
description: Claude cannot edit ~/.claude/settings.json enabledPlugins; the auto-mode classifier blocks it as self-modification of startup config
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 60632fa9-e3bb-4ae1-b906-ea0f0f727a6b
  modified: 2026-07-24T14:55:00.907Z
---

The auto-mode classifier blocks Claude from editing `~/.claude/settings.json` `enabledPlugins` (and likely other startup config keys), even after the user approves — it's treated as self-modification of agent startup config, outside project scope.

**Why:** It's a firm guardrail, not a per-call permission prompt; retrying after approval fails identically.

**How to apply:** Don't attempt the Edit and don't keep retrying. Give the user a ready-to-run `!`-prefix command (e.g. a python one-liner that loads the JSON, sets the key, writes it back) or the exact hand-edit, and remind them to restart Claude Code so `enabledPlugins` is re-read. Related: [[reference_sandbox_protected_paths]].
