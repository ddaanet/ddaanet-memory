---
name: cc-plugin-dirs-env-var
description: "loading a plugin via env instead of `--plugin-dir`: absolute `:`-separated paths only (`skipped …: a plugin folder here is an absolute local path or starts with ~`), inherited by hooks/children, same edit sensitivity; a symlinked root is `realpath`-resolved and still flags its target"
metadata:
  node_type: memory
  type: reference
  originSessionId: 5a7005eb-dc04-42af-a3e3-8a8e62461ed0
  modified: 2026-09-28T22:17:44.920Z
---

Verified by reading the Claude Code 2.1.283 bundle.

- **Format.** `CLAUDE_CODE_PLUGIN_DIRS` is split on `path.delimiter` (`:` on
  Unix), each entry trimmed. An entry is kept only if it is an absolute local
  path. Relative, `~`-prefixed and network/UNC entries are dropped with the
  warning `CLAUDE_CODE_PLUGIN_DIRS: skipped …: a plugin folder here is an
  absolute local path or starts with ~` (the wording reads inverted; the
  filter keeps absolute paths). It can also come from a settings `env` block.
- **Inheritance.** CC re-sets the variable in its own environment to the
  normalized list. Hooks and child processes (a nested `claude`) therefore see
  it, unlike a `--plugin-dir` flag. That is why a launcher shim exporting it
  can be checked from a `SessionStart` hook, and why a relaunch spawned from
  inside the process keeps it.
- **Same sensitivity as the flag.** Its entries join `--plugin-dir` and
  `--plugin-dir-no-mcp` in one inline-root list. Every path under an inline
  root is a sensitive file for Edit/Write, which is
  [[cc-plugin-dir-edits-sensitive]].
- **Symlinks do not escape.** Both the roots and the edited path go through
  `fs.realpathSync.native`, walking up to the nearest existing ancestor,
  before the is-under test. A symlinked root flags the tree it points to, so
  dogfooding without the prompts needs a real copy.
