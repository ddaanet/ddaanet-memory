---
name: reference_cc_worktree_deletion
description: "Deleted worktree cwd → CC's Bash shell falls back to main repo; ExitWorktree is a no-op for a worktree the session didn't create via EnterWorktree"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4de157b5-bece-4ad7-ab1f-e5d28e0e30a5
  modified: 2026-07-24T12:25:01.517Z
---

Two empirically-verified Claude Code behaviors, both observed in background
worktree session `5935efe7` (2026-07-02, transcript examined 2026-07-13) where
`$CLAUDE_PROJECT_DIR` was the worktree path itself (see
[[reference_cc_hook_no_worktree_field]]):

1. **When the Bash tool's cwd is deleted out from under a session** (here, the
   worktree was removed via `git worktree remove --force <self>`), the shell
   **falls back to the main repo root** — the post-deletion PreToolUse payload
   `cwd` became the main repo path, *not* the now-gone worktree path. A `cd` to
   the deleted path then silently no-ops (nowhere to go).

2. **`ExitWorktree` is a no-op for a worktree the current session did not create
   via `EnterWorktree`.** A background worktree session (`sessionKind:"bg"`) does
   not qualify, so `ExitWorktree` cannot be used to escape it — a hook or guard
   cannot rely on "document ExitWorktree as the exit" for the
   `$CLAUDE_PROJECT_DIR`-is-worktree shape.
