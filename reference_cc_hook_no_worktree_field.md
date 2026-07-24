---
name: reference_cc_hook_no_worktree_field
description: "Claude Code Bash hook payload has NO worktree field; detect worktrees from the filesystem, and mind the two $CLAUDE_PROJECT_DIR shapes"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 11d7dcf2-fb76-4e72-abee-c48d9a751bd8
  modified: 2026-07-24T12:24:55.354Z
---

The Claude Code `PreToolUse`/`PostToolUse(Bash)` hook stdin JSON has **no
`worktree` field**. Verified empirically on 2026-06-07 by capturing a real
payload from inside an `isolation: "worktree"` subagent (temporary capture hook
in `.claude/settings.local.json`, read back with the file Read tool to avoid
the capture overwriting itself).

Fields actually present: `session_id`, `transcript_path`, `cwd`,
`permission_mode`, `hook_event_name`, `tool_name`, `tool_input`, `tool_use_id`,
plus `effort` (top-level session) or `agent_id` + `agent_type` (subagents).

**`$CLAUDE_PROJECT_DIR` is NOT always the main repo root in a worktree.** Two
shapes to handle:

1. The common `isolation:"worktree"` subagent case keeps `$CLAUDE_PROJECT_DIR`
   at the original repo root — only `cwd` moves into the worktree.
2. A **background worktree session** (`sessionKind:"bg"`) sets
   `$CLAUDE_PROJECT_DIR` to the *worktree path itself* — verified from session
   `5935efe7` (2026-07-02). There `cwd == $CLAUDE_PROJECT_DIR == <worktree>`, so
   a walk-up-from-`cwd` worktree check finds nothing. Detect this shape by
   testing whether `$CLAUDE_PROJECT_DIR/.git` is itself a worktree `.git` file
   (`gitdir: <main>/.git/worktrees/<name>`) and slicing `<main>` before
   `/.git/worktrees/`.

To recognize a worktree from a hook, derive it from the **filesystem** — walk up
`cwd` to the ancestor whose `.git` is a file whose `gitdir:` resolves under
`$CLAUDE_PROJECT_DIR/.git`. Do not trust a payload field, and do not rely on the
`.claude/worktrees/<name>` path convention (worktrees can live at arbitrary
paths via `EnterWorktree path:` or a relocating `WorktreeCreate` hook).

A web/doc search during this work *claimed* a `worktree` field existed — it was
confabulated. Treat external API-schema claims as unverified until a real
payload is captured. Related: [[reference_cc_worktree_deletion]].
