---
name: feedback-claude-project-dir
description: Use CLAUDE_PROJECT_DIR not cwd from hook payload — cwd tracks shell working directory which can drift via /add-dir + cd
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9a80ffb9-0de3-409a-b1a9-d72343b3f28a
---

In hook scripts, use `CLAUDE_PROJECT_DIR` as the project root, not `cwd` from the hook payload JSON.

**Why:** `cwd` in hook payloads tracks the Bash tool's persistent shell working directory, not the project's configured directory. A `cd` command or `/add-dir` shifting focus to a second repo changes `cwd` for subsequent hook payloads, causing path guards to reference the wrong project. `CLAUDE_PROJECT_DIR` is set by CC to the project's configured root and is stable for the session.

**How to apply:** In any hook script that needs the project root: `cwd="${CLAUDE_PROJECT_DIR:-$PWD}"`. The `$PWD` fallback covers contexts where the env var is absent (e.g. direct testing without the full CC environment). The `jq .cwd` pattern from hook payloads should not be used as a fallback — it reintroduces the drift problem.
