---
name: cc-plugin-dir-edits-sensitive
description: "every Edit under a `--plugin-dir` root prompts (\"which is a sensitive file\", often no reason shown) despite acceptEdits and `Edit(...)` allow rules, or gets \"classifier gave no verdict (it skipped this action)\" in auto mode; load the plugin from a synced copy, a rename does nothing"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4cb08621-1ab2-4511-a5d3-02e1a2a20e76
  modified: 2026-09-26T14:03:53.478Z
---

Claude Code treats every path under an **inline plugin root** — a
`--plugin-dir` or `--plugin-dir-no-mcp` argument, or a
`CLAUDE_CODE_PLUGIN_DIRS` entry — as a sensitive file. Verified by reading the
CC 2.1.283 bundle: the Edit/Write permission function runs its path safety check
(`checkPathSafetyForAutoEdit`) **before** the acceptEdits auto-allow and before
settings allow rules, and that check returns `ask` for inline plugin roots. So:

- acceptEdits still prompts for each edit, and the dialog may show no reason.
- `Edit(/plugin/**)` in any settings scope is never consulted. Only session
  rules rooted at `/.claude/**` or `~/.claude/**` are checked earlier.
- In auto mode the same edits come back "The server-side auto mode classifier
  gave no verdict (it skipped this action)". That this is the same flag seen
  server-side is inferred, not verified.
- Subagents hit it too: the inline-plugin list is session-wide.

The name of the directory plays no part; no list contains `plugin`. What flags
it is being loaded. Paths with a `.claude` segment (e.g. a
`<dir>/.claude/skills` symlink spelling) are flagged separately, loaded or not.

**Why:** a session dogfooding its own plugin via `--plugin-dir` cannot edit
that plugin unattended — edify's `/build` stalled across five legs on it.

**How to apply:** when edits to a plugin tree prompt or get "no verdict"
despite allow rules, check the launch for `--plugin-dir` (a repo `.bin/claude`
shim, a `just claude` recipe) before touching permissions. The fix is to load
the plugin from a synced copy (e.g. `dist/plugin/`, refreshed at launch and on
edit) and edit the source tree, or to relaunch without the flag. Renaming the
directory does nothing. Related: [[cc-agent-discovery]],
[[classifier-denied-self-config]].
