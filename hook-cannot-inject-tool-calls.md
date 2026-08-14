---
name: hook-cannot-inject-tool-calls
description: "no `injectToolCall`/`forceRead`/`requestTool`/`forceToolUse` field exists — only block, `updatedInput`, `updatedToolOutput`, `additionalContext`; a large `additionalContext` is spilled to a file with only ~2KB inlined, and injected text still leaves `Edit` needing a `Read`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 9eca9443-3081-46d5-a1d8-7f881c0f4d41
  modified: 2026-08-14T08:46:08.725Z
---

No Claude Code hook can inject a `tool_use`, force a `Read`, or enqueue a tool
call. Only the agentic loop generates tool calls. Hooks can block
(`PreToolUse.permissionDecision`, `Stop`/`PostToolBatch` `decision: "block"`),
rewrite input (`updatedInput`), rewrite a result (`updatedToolOutput`), and
inject text (`additionalContext`) — nothing more.

Verified 2026-07-22 against the shipped binary
(`~/.local/share/claude/versions/2.1.217`), not only the docs: zero occurrences
of `injectToolCall`, `requestTool`, `forceRead`, `injectTool`, `forceToolUse`,
`requestedTools`, `injectedToolCalls`. The four `triggerRead` hits are Node
stream internals (`stream._read = triggerRead`). Controls present in the same
grep: `additionalContext` 180, `updatedInput` 249, `permissionDecision` 34,
`PostToolBatch` 46, `InstructionsLoaded` 27, `systemMessage` 54.

**How to apply:** when you want file contents in the model's context
deterministically, don't try to make the agent Read them — have the hook read
them and emit the bytes as `additionalContext`. That is strictly better than a
forced Read: one round trip instead of two, no tool-permission surface, and the
selection stays in a script rather than in agent judgement.
`additionalContext` is honoured at `PreToolUse`
as well as `PostToolUse` and arrives even when the tool call fails
([[hook-output-channels]]), so a deny-reason and the injected
bodies can land in the same turn.

**`updatedToolOutput` is validated against the tool's own output shape**, and a
mismatch is rejected with `PostToolUse hook returned updatedToolOutput that does
not match <tool>'s output shape` (string present in 2.1.232). So rewriting what
a tool reported is not a free-text channel: a hook that corrects, say, an `Edit`
result has to reproduce that tool's result object, not substitute prose for it.
Budget for establishing the shape before designing around this channel;
`additionalContext` carries a correction with no shape contract at all, at the
cost of leaving the original result in context beside it.

**Two limits make emitted bytes the wrong choice when the agent must *edit*
what it is shown.** A large `additionalContext` is not delivered whole: at
15.6KB the harness wrote it to a `tool-results/…-additionalContext.txt` file
and inlined only a ~2KB preview, so a hook emitting several file bodies
silently delivers a fraction of them unless the agent notices the pointer and
reads that file. And injected text does not register in the file-read ledger —
the ledger is keyed off actual `Read` calls — so `Edit` on a file whose bytes
arrived by `additionalContext` still fails until the agent Reads it, paying for
the content twice. There is no supported way to prime the ledger. Where the
recalled file is going to be edited, emit a directive naming the exact `Read`
calls instead of the bytes: two round trips, but no truncation and the file
lands editable.

The complement — *obliging* the agent to reach the checkpoint at all — is a
separate mechanism: a `PreToolUse` deny on the first durable write of an
episode is self-scoping (no edit, no gate), where a `Stop` block would fire on
every turn including conversational ones.

`InstructionsLoaded` fires when `CLAUDE.md` and `.claude/rules/*.md` load
(matchers `session_start`, `nested_traversal`, `path_glob_match`, `include`,
`compact`) but is side-effects-only: no decision control, exit code ignored.
