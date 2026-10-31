---
name: marketplace-dir-in-direnv
description: "MARKETPLACE_DIR is already exported via ~/.envrc (direnv) — don't manually prefix it when running `just release`"
metadata:
  type: feedback
  originSessionId: 30f0ab5a-17b0-448d-aafc-df8c311958bd
---

When running `just release` (or any other recipe that consumes `MARKETPLACE_DIR`) on this user's machine, **do not** prepend `MARKETPLACE_DIR=...` to the command.

**Why:** The user keeps `MARKETPLACE_DIR=/Users/david/code/claude-plugins` exported globally via `~/.envrc` (`~/.envrc` is whitelisted in direnv config). Bash sessions launched by Claude Code inherit it. Manually re-setting it on the command line is noise; the user flagged it as unnecessary.

**How to apply:** When invoking `just release patch --yes` (or similar), just run the command. If the recipe complains about a missing env var, check `echo $MARKETPLACE_DIR` first before adding a prefix — direnv may have failed to load for some other reason.
