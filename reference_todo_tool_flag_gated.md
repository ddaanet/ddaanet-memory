---
name: reference-todo-tool-flag-gated
description: "`TodoWrite` has no tool definition in 2.1.217; `TaskCreate`/`TaskGet`/`TaskList`/`TaskUpdate` replace it, gated by server-side `tengu_vellum_ash` — never name a todo tool in a plugin step"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4cfddafb-441d-4f27-b514-ed3622e8fc11
  modified: 2026-07-22T17:06:58.654Z
---

Verified against the Claude Code 2.1.217 bundle on 2026-07-22, after the todo
tool was observed **absent from a live session** with no local config change.

`TodoWrite` still appears 13 times but has **no tool definition** — no
`userFacingName(){return"TodoWrite"}`. What survives is vestigial: a bare
constant (`var Nj="TodoWrite"`), membership in timeout/permission/eval
name-sets, the stale nag copy ("consider using the TodoWrite tool"), and a scan
that reads *historical* `TodoWrite` entries out of old transcripts.

The replacement is a task-list family with real definitions —
`TaskCreate` ("create a task in the task list"), `TaskGet`, `TaskList`,
`TaskUpdate` — all `shouldDefer: true`, so they surface via ToolSearch rather
than the upfront tool list. Note this is CRUD (per-item `TaskUpdate`) where
`TodoWrite` rewrote the whole array. Gating:

```js
isEnabled(){ return bL() && !TZ() }
function bL(){ if (Z.CLAUDE_CODE_ENABLE_TASKS === !1) return !1; return !0 }
function TZ(){ let e = Ke("tengu_vellum_ash", []); … return e.some(r => Ii().includes(r)) }
```

`bL()` is on by default; `TZ()` is a **server-side killswitch**. Neither family
was reachable in the observing session with `CLAUDE_CODE_ENABLE_TASKS` unset.
Separately, `OFs()` gates the "you haven't used it lately" nudge off
`CLAUDE_CODE_TODO_REMINDER_MODE` / `tengu_soft_slate_nudge`.

Two consequences. **Never build a plugin step that names a todo tool** — the
affordance can differ between two sessions on the same binary with the same
settings, and the unofficial changelog (claudefa.st) lagged ~90 versions behind
and had no entry at all. And **transcript scraping for todo state is a trap**:
Claude Code's own nag keys on `tool_use` with `name === "TodoWrite"` plus
`attachment.type === "todo_reminder"`, and now reads a dead string — the same
JSONL-coupling hazard as [[reference_session_jsonl_schema]].

Applied in [[project_handoff_todo_ledger]]: the file is the ledger, whatever
tracker happens to be live is a cache, and nothing in the skill body names one.
