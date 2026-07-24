---
name: feedback-uv-direnv-venv
description: "don't use `uv run` with Claude (sandbox/permission pain); use `uv sync` only + direnv-activated venv on PATH"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 53e0f01b-8a6c-42ef-9bb1-8b971e31b012
---

David gave up on `uv run` (and `uvx`) under Claude Code — the sandbox and
permission complexity around uv's cache (`~/.cache/uv`) isn't worth it.

**Why:** every `uv run` touches `~/.cache/uv`, which the Bash sandbox
blocks; threading that through settings/permissions for each project is
more friction than it buys.

**How to apply:** use `uv` only for `uv sync` (materialize `.venv` +
`uv.lock`, a manual/occasional step). For everything else rely on
**direnv** to activate the venv: `.envrc` exports `VIRTUAL_ENV=$PWD/.venv`
and prepends `$VIRTUAL_ENV/bin` to `PATH` directly. Test/build recipes
then call bare `pytest` (etc.) off PATH — no `uv run` wrapper. See
[[project-test-suite-framework-port]].
