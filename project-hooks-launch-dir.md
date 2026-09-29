---
name: project-hooks-launch-dir
description: "a hook registered in the repo root's `.claude/settings.json` (version-guard, a dogfood guard, any `PreToolUse`/`SessionStart`) never fires, no error: `claude` was launched from a subdirectory — project settings are read from the launch directory only, not the git root"
metadata:
  node_type: memory
  type: reference
  originSessionId: 601f5335-05d0-4e40-8f7e-ae3d1b4ac311
  modified: 2026-09-29T20:45:45.996Z
---

Claude Code reads project settings, hooks included, from the directory
`claude` is launched in. It does not walk up to the git root. A session started
in `<repo>/sub/dir` runs none of the hooks in `<repo>/.claude/settings.json`,
not even an inline `echo` hook, and nothing reports the absence: the session
behaves normally otherwise. Probed on CC 2.1.284 with unsandboxed
`claude -p` runs from a scratch repo's root and from a nested subdirectory; from
the root every hook fired, with `CLAUDE_PROJECT_DIR`, `$PWD` and the payload
`cwd` all naming the root.

**Consequence:** every guard wired as a project hook — `claude-plugin-dev`'s
version-guard in each consumer, a copy guard, a `SessionStart` warning — is
absent from a subdirectory launch. A launcher that loads a plugin by absolute
path (`--plugin-dir`, `CLAUDE_CODE_PLUGIN_DIRS`) still loads it there, so the
plugin appears to work while its safety hooks are off.

**How to apply:** when a project hook "does nothing", check the launch
directory before debugging the hook. Document "launch from the repo root" for
any design that relies on project hooks; a message printed by a launch shim
before `exec claude` is not a remedy, since the TUI hides it.
