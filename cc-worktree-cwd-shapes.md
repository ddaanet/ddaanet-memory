---
name: cc-worktree-cwd-shapes
description: "a `sessionKind:\"bg\"` worktree session sets `$CLAUDE_PROJECT_DIR` to the worktree path itself, so a walk-up-from-`cwd` check finds nothing; a deleted worktree cwd falls back to the main repo (`cd` there silently no-ops); `ExitWorktree` no-ops for a worktree the session did not create via `EnterWorktree`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 11d7dcf2-fb76-4e72-abee-c48d9a751bd8
  modified: 2026-08-26T07:57:34.599Z
---

What a git worktree does to the `cwd` and `$CLAUDE_PROJECT_DIR` a Claude Code
hook or guard sees, and how the escape hatch behaves. Established from a real
`PreToolUse(Bash)` payload captured 2026-06-07 inside an
`isolation:"worktree"` subagent (temporary capture hook in
`.claude/settings.local.json`, read back with the Read tool so the capture could
not overwrite itself), and from background worktree session `5935efe7`
(2026-07-02, transcript examined 2026-07-13).

**`$CLAUDE_PROJECT_DIR` is NOT always the main repo root.** Two shapes to
handle:

1. The common `isolation:"worktree"` subagent keeps `$CLAUDE_PROJECT_DIR` at the
   original repo root — only `cwd` moves into the worktree.
2. A **background worktree session** (`sessionKind:"bg"`) sets
   `$CLAUDE_PROJECT_DIR` to the *worktree path itself*. There
   `cwd == $CLAUDE_PROJECT_DIR == <worktree>`, so a walk-up-from-`cwd` worktree
   check finds nothing. Detect this shape by testing whether
   `$CLAUDE_PROJECT_DIR/.git` is itself a worktree `.git` file
   (`gitdir: <main>/.git/worktrees/<name>`) and slicing `<main>` before
   `/.git/worktrees/`.

**A deleted cwd falls back to the main repo.** When the Bash tool's cwd is
removed out from under a session (here `git worktree remove --force <self>`),
the shell falls back to the main repo root: the post-deletion `PreToolUse`
payload `cwd` is the main repo path, not the now-gone worktree path. A `cd` to
the deleted path then silently no-ops — there is nowhere to go.

**`ExitWorktree` is a no-op for a worktree the current session did not create
via `EnterWorktree`.** A `sessionKind:"bg"` worktree session does not qualify,
so `ExitWorktree` cannot be used to escape it — a hook or guard must not
document "leave with `ExitWorktree`" as the exit for the
`$CLAUDE_PROJECT_DIR`-is-the-worktree shape. Where the session *did* enter
through `EnterWorktree`, `ExitWorktree` restores both cwd and `projectRoot`; see
[[cc-worktree-memory-freeze]].

**How to apply:** recognize a worktree from the **filesystem** — walk up `cwd`
to the ancestor whose `.git` is a file whose `gitdir:` resolves under
`$CLAUDE_PROJECT_DIR/.git`, then test `$CLAUDE_PROJECT_DIR` itself for shape 2.
Hook stdin carries no worktree-root field at all ([[hook-input-schema]]), and
the `.claude/worktrees/<name>` path convention is not a reliable substitute —
worktrees can live at arbitrary paths via `EnterWorktree path:` or a relocating
`WorktreeCreate` hook. A doc/web search during this work claimed a `worktree`
payload field exists; it was confabulated. Treat an external API-schema claim as
unverified until a real payload is captured.
