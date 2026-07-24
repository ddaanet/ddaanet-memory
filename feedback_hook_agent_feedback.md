---
name: Hook output channels — user vs agent
description: systemMessage in hook JSON is user-facing only; the agent needs additionalContext (or stderr) to receive feedback about what a hook did
type: feedback
originSessionId: ca091749-f579-4ae6-8c1d-5dc525edcad5
---
A hook that emits only `{"systemMessage": "..."}` is invisible to the
agent — that field surfaces in the UI/transcript for the user, not in
the agent's input context. If the agent needs to know what the hook
did (so it doesn't redundantly verify with `ls`/`cat`/etc.), the hook
must use an agent-facing channel:

- `UserPromptSubmit`: `hookSpecificOutput.additionalContext` (string
  injected into the prompt context).
- `PreToolUse`: `hookSpecificOutput.additionalContext` or
  `permissionDecisionReason` (on deny). stderr+exit 2 also surfaces.
- `PostToolUse`: stderr+exit 2 to feed back; `additionalContext`
  available for some events.

**Why:** observed live: the handoff plugin's `prompt-pre-hook.sh` and
`skill-pre-hook.sh` emitted only `systemMessage` when wiping prior
handoff files. The wipe ran correctly but the agent received no
notification, so the agent ran `ls .claude/handoff-task.md
.claude/handoff.md` to verify — wasted work the hook was designed to
avoid.

**How to apply:** when designing or reviewing hook output, decide
explicitly who the audience is. User-only → `systemMessage`.
Agent-only → `additionalContext`. Both → emit both fields. Don't
assume `systemMessage` is general-purpose.
