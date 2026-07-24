---
name: reference-hook-input-schema
description: "base stdin = session_id/transcript_path/cwd/permission_mode?/agent_id?; every event incl SessionStart carries cwd; no worktree-root field; CwdChanged is observational (watchPaths+systemMessage only, no block/additionalContext)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7a891994-6ca1-4ea3-a503-1259d89a76e2
---

Verified against the Claude Code 2.1.168 bundle (zod schemas). The base hook
input `Sw` = `{ session_id, transcript_path, cwd, permission_mode?, agent_id? }`,
composed via `Sw().and(...)` into **every** hook event — so SessionStart,
PreToolUse, PostToolUse, UserPromptSubmit, etc. all carry `cwd` (a required
string). There is **no worktree-root field** in hook stdin; `cwd` is the only
directory context, and it is drift-prone (moves with `cd` / `/add-dir`).

Newer hook events also exist: `WorktreeCreate {name}`,
`WorktreeRemove {worktree_path}`, `CwdChanged {old_cwd,new_cwd}`,
`FileChanged {file_path,event}`. **`CwdChanged` is observational only** — its
dispatcher (`g39`) reads back just `watchPaths[]` + `systemMessage`; it honors
**no** `additionalContext`, **no** `decision`/block, **no** `permissionDecision`,
and runs async fire-and-forget (errors swallowed). It cannot gate a `cd` or feed
the agent. By contrast `WorktreeCreate`'s dispatcher *does* honor
`decision==="block"`.

**How to apply:** when a hook needs the project/worktree root, derive it from the
on-disk `.git` linkage starting at `.cwd` — don't expect a root field in the
payload, and don't trust raw `.cwd` as the root. See
[[feedback-claude-project-dir]] and handoff's `worktree_root.py`.
