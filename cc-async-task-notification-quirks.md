---
name: cc-async-task-notification-quirks
description: "`TaskOutput(block=false)` on a running `local_agent`/fork can dump the raw JSONL transcript; a stale task-id can refire \"completed\" with unrelated content — trust the agent's own report, not a late notification"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1228fd49-f82a-49ea-98e6-bf3c011f0e9d
  modified: 2026-08-10T17:07:22.953Z
---

`TaskOutput(task_id, block=false)` on a still-running `local_agent` task can
return the raw JSONL transcript tail (tool calls, diffs, full file bodies) in
the tool result itself, not just a status line — despite the tool's own
guidance to avoid reading a `local_agent`'s `.output` file directly. Checking
on a running fork this way risks the same context blowup that guidance warns
against. Prefer waiting for the `<task-notification>` instead of polling.

A `<task-notification>` can also refire for the same `task-id` well after it
first reported `completed`, carrying content unrelated to the original task.
Cause, confirmed: a background agent's own session is independently
interactive — my human partner can send it a message directly, outside the
parent conversation, and that resume produces a new notification answering
whatever was asked there, not the originally delegated task. Treat only the
content an agent sent via its own reply/SendMessage for its delegated task
as authoritative; a later notification on an already-reported task-id may
be answering something else entirely.
