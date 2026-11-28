---
name: vibe-headless-probe-isolated
description: "live-probing a Vibe plugin headless without touching ~/.vibe or the checkout: VIBE_HOME relocates the whole Vibe home, symlink .env into it, run vibe -p unsandboxed since the command sandbox denies api.mistral.ai"
metadata:
  node_type: memory
  type: reference
  originSessionId: cfc6131a-9f10-4b5f-b01f-23224c63778f
  modified: 2026-11-28T18:22:59.322Z
---

Measured 2026-11-28, Vibe 3.0.1, from a Claude Code session.

- `VIBE_HOME=<dir>` relocates every Vibe path (plugins, config, `.env`,
  trusted folders, logs; `vibe/core/paths/_vibe_home.py`). Copy the plugin
  under test into `<dir>/plugins/<name>` and its dependencies beside it
  (gitlore's native plugin, for a boundary that reaches `gitlore:commit`),
  copy `config.toml`, and symlink `~/.vibe/.env` rather than copying the key.
- Run in a `git clone` of the consumer under `$TMPDIR`, so the skill's
  writes land there: `vibe --trust --auto-approve --max-price 1 --time-limit 420
  --output-dir <out> -p '/<ns>:<skill> …'`. `--trust` is needed or the
  project's AGENTS.md and `.vibe/hooks.toml` are skipped.
- The Claude Code command sandbox denies `api.mistral.ai`; the run needs
  `dangerouslyDisableSandbox`.
- `git clone --recurse-submodules` of a repo whose memory gitlink is not on
  the remote leaves the memory submodule with every file staged as deleted,
  so a boundary probe sees "dirty memory" that it did not cause. Read it as a
  clone artifact, not a skill bug.
- `<out>/session/journal/*.jsonl` holds the tool calls; `export.json` the
  cost and stop reason.

Related: [[vibe-native-plugins]].
