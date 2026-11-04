---
name: marketplace-dir-in-direnv
description: "don't prefix `MARKETPLACE_DIR=...` when running `just release`; `~/code/.envrc` already exports it"
metadata:
  type: feedback
  originSessionId: 30f0ab5a-17b0-448d-aafc-df8c311958bd
  modified: 2026-11-01T16:03:29.322Z
---

When running `just release` (or any other recipe that consumes `MARKETPLACE_DIR`) on this user's machine, **do not** prepend `MARKETPLACE_DIR=...` to the command.

**Why:** The user keeps `MARKETPLACE_DIR=/Users/david/code/claude-plugins` exported via `~/code/.envrc`, which each repo's `.envrc` pulls in with `source_up_if_exists`. Bash sessions launched by Claude Code inherit it. Manually re-setting it on the command line is noise; the user flagged it as unnecessary.

**How to apply:** When invoking `just release patch --yes` (or similar), just run the command. If the recipe complains about a missing env var, check `echo $MARKETPLACE_DIR` first before adding a prefix — direnv may have failed to load for some other reason.
