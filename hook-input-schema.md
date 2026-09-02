---
name: hook-input-schema
description: "`tool_input.file_path` is NOT always absolute; base stdin = `session_id`/`transcript_path`/`cwd` + optional `prompt_id`/`permission_mode`/`agent_id`/`agent_type`/`model`/`session_title`; no worktree-root field; `CwdChanged` observational; `WorktreeCreate` provides a path, doesn't gate; `PostToolBatch` fires per tool batch (twice in one turn is normal); NO event carries token counts (context size comes from transcript `usage`); `additionalContext` lands mid-turn, `continue:false` halts one"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7a891994-6ca1-4ea3-a503-1259d89a76e2
  modified: 2026-08-01T14:37:21.679Z
---

Verified against the Claude Code bundle (zod schemas) — first on 2.1.168,
re-verified on **2.1.219, 2026-07-24**. This is the complement of what
`plugin-dev:hook-development` documents, not a copy of it: that skill covers
`session_id`/`transcript_path`/`cwd`/`permission_mode` and the classic events,
and carries none of the optional base fields, no `SessionStart` `source`
values, and none of the extended event union below. It ships in a marketplace
plugin, so the gap cannot be closed at its source.

The base hook input is composed via `.and(...)` into **every** hook event, so
SessionStart, PreToolUse, PostToolUse, UserPromptSubmit, etc. all carry it:

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

`tool_input.file_path` on `Write`/`Edit` is **not** always absolute, whatever
the tool's own contract says. `PreToolUse` sees what the model emitted, before
any harness normalization, so a path guard that assumes a leading `/` — a
`*/<segment>/*` glob, a prefix comparison — silently misses the relative form.
Measured over a local transcript corpus 2026-08-27: 131 of ~4100 values were
repo-root-relative (`plans/x.md`, `scripts/install.sh`, `START.md`). A hook
resolving one has only its own process cwd to resolve it against.

**`PostToolBatch` fires once per tool batch, not once per user turn.** A batch is
one assistant message's worth of calls, so a turn in which the agent takes three
batches fires it three times. Observed 2026-07-27 in gitlore: a `sed -i` in one
message fired the batch hooks with that message's calls, and the very next
message fired them again. A hook that stashes a pre-state at `PreToolUse` and
consumes it at `PostToolBatch` therefore has a *per-batch* baseline — treating it
as per-turn overstates how long a stash lives.

**No hook event carries token counts**, and `PostToolBatch` is the only event
that fires *inside* a turn. Verified 2026-07-31 on 2.1.220 by nested `claude -p`
runs. There is no context percentage or window size on any payload, so context
size comes from the transcript: on the newest entry carrying `.message.usage`,
`input_tokens + cache_creation_input_tokens + cache_read_input_tokens` is the
prompt sent for that call (accurate after a compaction). Take the *last* such
entry rather than summing — one API response emits several JSONL entries sharing
a `message.id`, each repeating the same `usage`. Inside a subagent
`transcript_path` points at the **parent**; the subagent's own usage is at
`<session-dir>/subagents/agent-<agent_id>.jsonl`, so a naive read measures the
wrong context. `PostToolBatch` stdin also carries an undocumented `tool_calls[]`
(`tool_name`/`tool_input`/`tool_use_id`/`tool_response`, already resolved) —
absent from the binary's own schema doc-strings, visible only in a live payload.

**`hookSpecificOutput.additionalContext` reaches the model on the next API call
of the same turn**, and **`continue: false` halts a running turn** — the batched
calls still run and their results land in the transcript, then the turn ends with
no further API call and no final assistant text (`--debug hooks` names it
`requested preventContinuation`). `decision: "block"` halts too, by a different
internal path. Together these are the only levers a mid-turn hook has: inject,
message, or halt — see [[hook-output-channels]].

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

**A `SessionStart(clear)` payload describes the new session, and its transcript
already exists when the hook runs.** `transcript_path` names the post-`/clear`
session's file, on disk by hook time — so a `clear` hook can baseline a
transcript count and confirm a prompt submitted into the fresh session, exactly
as a `SessionStart(compact)` hook can. That payload carries no `model` where
`startup` does; the field is optional in the base schema, so read it
defensively (2.1.219).

**How to apply:** when a hook needs the project/worktree root, derive it from the
on-disk `.git` linkage starting at `.cwd` — don't expect a root field in the
payload, and don't trust raw `.cwd` as the root. See
[[claude-project-dir]] and handoff's `worktree_root.py`.
