---
name: uv-direnv-venv
description: "no `uv run` under Claude; `uv sync` only + direnv exports VIRTUAL_ENV & PATH prefix; recipes call bare `pytest`; the venv python3 shadows the system one, so every module scripts on PATH probe for goes in the dev group (`need … PyYAML`); a symlink to the venv's python3 resolves to the base interpreter and loses site-packages — borrow it with an `exec` wrapper and probe through the shim"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 53e0f01b-8a6c-42ef-9bb1-8b971e31b012
  modified: 2026-09-25T21:48:17.313Z
---

My human partner gave up on `uv run` (and `uvx`) under Claude Code — the
sandbox and permission complexity around uv's cache (`~/.cache/uv`) isn't
worth it.

**Why:** every `uv run` touches `~/.cache/uv`, which the Bash sandbox
blocks; threading that through settings/permissions for each project is
more friction than it buys.

**How to apply:** use `uv` only for `uv sync` (materialize `.venv` +
`uv.lock`, a manual/occasional step). For everything else rely on
**direnv** to activate the venv: `.envrc` exports `VIRTUAL_ENV=$PWD/.venv`
and prepends `$VIRTUAL_ENV/bin` to `PATH` directly. Test/build recipes
then call bare `pytest` (etc.) off PATH — no `uv run` wrapper.

**The venv's `python3` shadows the system one for every script on PATH**, so
the dev group must carry every module those scripts probe for, not only the
tools the recipes call. In gitlore, adding a venv for one tool (rumdl)
turned the hook-wiring suite red — `need mikefarah/yq or python3 with
PyYAML` — because `wire-*.sh` run whatever `python3` is on PATH and the
venv's had no `yaml`. The system python having PyYAML was an undeclared
dependency of the gate; `pyyaml` in the dev group is what made it
reproducible. Calling the tool by path instead keeps the hidden dependency.

**A symlink to the venv's `python3` is not the venv.** A venv interpreter
locates its `pyvenv.cfg`, and through it its site-packages, beside the path it
was invoked as; reached through a link in another directory it resolves to the
base interpreter, `sys.prefix` becomes the base install, and `import yaml`
fails. A harness that borrows the venv's python onto a narrowed `PATH` needs an
`exec` wrapper, not `ln -s` — and has to probe *through* the shim. gitlore's
macOS runner probed the absolute path, where the import works, linked it, and
reported a successful borrow the suites could not use: forty-two failures whose
stated cause was a missing module.
