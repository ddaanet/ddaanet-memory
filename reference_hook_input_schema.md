---
name: reference-hook-input-schema
description: "base stdin=`session_id`/`transcript_path`/`cwd`+optional `prompt_id`/`permission_mode`/`agent_id`/`agent_type`/`model`/`session_title`; no worktree-root field; `CwdChanged`=observational; `WorktreeCreate` gives a path, doesn't gate; `PostToolBatch` fires per batch (twice/turn normal)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7a891994-6ca1-4ea3-a503-1259d89a76e2
  modified: 2026-07-30T15:18:10.054Z
---

Verified against the Claude Code bundle (zod schemas) — first on 2.1.168,
re-verified on **2.1.219, 2026-07-24**. The base hook input is composed via
`.and(...)` into **every** hook event, so SessionStart, PreToolUse, PostToolUse,
UserPromptSubmit, etc. all carry it:

```
{ session_id, transcript_path, cwd,
  prompt_id?, permission_mode?, agent_id?, agent_type?, model?, session_title? }
```

`cwd` is a required string. There is **no worktree-root field** in hook stdin;
`cwd` is the only directory context, and it is drift-prone (moves with `cd` /
`/add-dir`). `agent_id` is present only when the hook fires from within a
subagent; `agent_type` also appears on the main thread of an `--agent` session.
`prompt_id` correlates every event back to the user prompt that started the turn
(same value as the OTel `prompt.id` attribute), absent before the first prompt.

Beyond the classic events the input union covers `WorktreeCreate {name}`,
`WorktreeRemove {worktree_path}`, `CwdChanged {old_cwd,new_cwd}`,
`FileChanged {file_path,event:"change"|"add"|"unlink"}`,
`DirectoryAdded {directory,source}`, `InstructionsLoaded`, `ConfigChange`,
`PostToolBatch`, `PostToolUseFailure`, `PostCompact`, `Setup`, `SubagentStart`,
`StopFailure`, `PermissionRequest`, `PermissionDenied`, `MessageDisplay`,
`Elicitation`/`ElicitationResult`, `TaskCreated`/`TaskCompleted`.

**`PostToolBatch` fires once per tool batch, not once per user turn.** A batch is
one assistant message's worth of calls, so a turn in which the agent takes three
batches fires it three times. Observed 2026-07-27 in gitlore: a `sed -i` in one
message fired the batch hooks with that message's calls, and the very next
message fired them again. A hook that stashes a pre-state at `PreToolUse` and
consumes it at `PostToolBatch` therefore has a *per-batch* baseline — treating it
as per-turn overstates how long a stash lives.

**`CwdChanged` is observational only** — its `hookSpecificOutput` schema declares
nothing but `watchPaths`, and its dispatcher reads back only `watchPaths[]` +
`systemMessage`. It honors **no** `additionalContext`, **no** `decision`/block,
**no** `permissionDecision`. It cannot gate a `cd` or feed the agent.
`FileChanged` shares that dispatcher and the same restriction.

**`WorktreeCreate` is a path *provider*, not a decision gate.** Its dispatcher
takes the worktree path from the hook's stdout (or from
`hookSpecificOutput.worktreePath`, which its schema requires) and throws when no
hook ran, when every hook exited non-zero, or when none returned a path. A hook
refuses by failing — success is `exit status === 0`; `decision:"block"` plays no
part.

**`/clear` mints a new session, and `SessionStart(clear)` reports it.** Measured
in a tmux pane 2026-07-30 with a hook logging the raw payload: the startup
session was one id, the `clear` payload carried a different one, and a second
`<session-id>.jsonl` appeared beside the first under the project directory. Its
`transcript_path` is the **new** session's, and that file already exists when
the hook runs — so a `SessionStart(clear)` hook can baseline a transcript count
and confirm a prompt submitted into the fresh session, exactly as a
`SessionStart(compact)` hook can. (That payload carried no `model`, which
`startup` did; the field is optional in the base schema, so read it defensively.)

**How to apply:** when a hook needs the project/worktree root, derive it from the
on-disk `.git` linkage starting at `.cwd` — don't expect a root field in the
payload, and don't trust raw `.cwd` as the root. See
[[feedback-claude-project-dir]] and handoff's `worktree_root.py`.
