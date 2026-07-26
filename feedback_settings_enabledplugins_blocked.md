---
name: feedback_settings_enabledplugins_blocked
description: "classifier blocks Claude editing enabledPlugins even after approval; give the user a !-command"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 60632fa9-e3bb-4ae1-b906-ea0f0f727a6b
  modified: 2026-07-25T15:07:08.371Z
---

The auto-mode classifier blocks Claude from editing `~/.claude/settings.json` `enabledPlugins` (and likely other startup config keys), even after the user approves — it's treated as self-modification of agent startup config, outside project scope.

**Why:** It's a firm guardrail, not a per-call permission prompt; retrying after approval fails identically.

**How to apply:** Don't attempt the Edit and don't keep retrying. Give the user a ready-to-run `!`-prefix command (e.g. a python one-liner that loads the JSON, sets the key, writes it back) or the exact hand-edit, and remind them to restart Claude Code so `enabledPlugins` is re-read. Related: [[reference-sandbox-effects]] (the harness-mounted config paths the sandbox blocks — a different mechanism from the classifier).
