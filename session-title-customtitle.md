---
name: session-title-customtitle
description: "`hookSpecificOutput.sessionTitle` at `SessionStart` (in-memory) and `UserPromptSubmit` (persists) ONLY; storage is a last-wins `custom-title` jsonl entry; `/rename`, `-n`, ctrl+R write the same"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a7e28213-e5b0-4230-a6b4-882fe813e66e
  modified: 2026-07-24T19:23:15.135Z
---

A Claude Code session's title is the `customTitle`, persisted as a line in the session transcript `.jsonl`:

```json
{"type":"custom-title","customTitle":"my title","sessionId":"<id>","uuid":"<uuid>","timestamp":"<iso>"}
```

`/rename <name>` (alias `/name`) writes exactly this (appends the entry, sets in-memory `currentSessionTitle`, emits `tengu_session_renamed` with a `source`). On load/resume CC derives the title by scanning the transcript for the last `type:"custom-title"` entry (last-wins). CLI flag `-n, --name <name>` sets it at startup; picker rename is Ctrl+R. A sibling `type:"ai-title"` entry carries CC's own generated title; `customTitle` takes precedence over it.

**A hook sets the title with `hookSpecificOutput.sessionTitle`** (a string), available on **`SessionStart` and `UserPromptSubmit` only** — no other event's output schema declares the field (`PostToolUse` carries `additionalContext` / `updatedToolOutput` / `updatedMCPToolOutput`). The value has control characters stripped and is truncated to 200 chars; it is ignored when it matches the current title and when the hook fires from a subagent or child session. The two events differ in durability: `UserPromptSubmit` **persists** it — appends the `custom-title` entry and updates the live title — while `SessionStart` only **caches** it into the in-memory `currentSessionTitle`, which the session-storage flush later writes out as a `custom-title` entry. Hooks also *receive* the current title: `session_title` is a base hook-input field (see [[hook-input-schema]]).

Every hook gets `session_id` + `transcript_path` on stdin, so a hook script can also append a `custom-title` line to `transcript_path` itself; that route takes effect on the next read/resume rather than live, and is only worth it from an event without the `sessionTitle` field. Transcript path follows the [[cc-project-dir-encoding]] mangling under `~/.claude/projects/<encoded>/<session-id>.jsonl`. Transcript mechanism first verified on CC 2.1.150, 2026-05-25; re-verified plus the `sessionTitle` hook field on CC 2.1.219 by binary inspection, 2026-07-24.
