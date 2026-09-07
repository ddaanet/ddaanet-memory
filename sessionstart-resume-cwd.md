---
name: sessionstart-resume-cwd
description: "two hooks in one session naming different repos, or a session-keyed file naming a repo the session was never in: `SessionStart(resume)` fires with the RESUMING process's cwd before the harness moves the session into the resumed project dir, so never park a session-start snapshot of an environment fact"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8ce39f20-8c13-4bc3-a36c-76d6c3145b0a
  modified: 2026-09-02T22:12:34.962Z
---

Resuming a conversation belonging to repo A from a pane running in repo B fires
`SessionStart(resume)` with **B**'s cwd (`CC 2.1.220`). The harness moves the
session into the resumed conversation's project dir *after* that hook runs. So:

- `SessionStart` sees B.
- Everything from the first `UserPromptSubmit` onward sees A — transcript
  entries, `CLAUDE_PROJECT_DIR`, and every per-call hook.
- The resumed conversation's transcript and project dir stay A throughout, so
  every signal except the `SessionStart` sample agrees.

A session id with no transcript is not evidence of a bug: the `/resume` picker
runs in a session of its own, which may be minted, used only to pick, and
abandoned, leaving hook-written state behind an empty transcript (or one
holding only a `custom-title` entry).

**Consequence.** A hook that samples an environment fact once at `SessionStart`
and parks it for later use is sampling at the one moment the cwd does not belong
to the session. The wrong value is silent — nothing compares it against the
per-call resolution — until some consumer refuses. Two hooks in one session
reporting different repos is the diagnostic signature, and it is legitimate
rather than a race.

**What to do instead:** resolve per call, in the hook that needs it, from that
hook's own payload `cwd`. If a value must reach a context that cannot resolve it
(the agent's Bash — see the `plugin-craft:skill-authoring` skill), inject it at call time with
`PreToolUse` + `updatedInput` rather than parking a session-start snapshot.

Distinct from cwd **drift**, where the session genuinely leaves its launch repo:
drift is reported by the per-call resolution and is observable. This is not —
the session never moves, only the sample is foreign.
