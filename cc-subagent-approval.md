---
name: cc-subagent-approval
description: "`No agent named 'X' is currently addressable`: a one-shot sub-agent has no parent to `SendMessage`; a bare `idle_notification` is not a report"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 39084536-e084-480e-981f-18bb2e894d63
  modified: 2026-08-12T08:22:30.116Z
---

When a parent dispatches a sub-agent via `Agent({subagent_type, prompt})` (one-shot), the sub-agent has **no addressable parent** for `SendMessage`. Attempting `SendMessage to: "parent"` or `"team-lead"` returns `No agent named 'X' is currently addressable`. A sub-agent that needs parent approval and tries to SendMessage will fail the call, and (without a strict "stop on failure" clause) will rationalize proceeding — exactly what the gate was meant to prevent.

**Correct pattern: two turns.**

1. **Turn 1 (dispatch):** parent calls `Agent({subagent_type, prompt})`. Sub-agent does prep work (no destructive ops), then **returns** the approval request as its final message. Parent captures the returned `agentId`.
2. **Turn 2 (resume):** parent reads the return message, decides, then `SendMessage({to: <agentId>, message: "approved" | "rejected: <reason>"})`. The sub-agent resumes from the SendMessage and either runs the destructive op (on approval) or re-does the prep with feedback (on rejection).

This works because SendMessage *to a known agentId* succeeds even after the sub-agent's first task completes; the harness resumes from transcript.

**Contract must include:** "If you are resumed without a clear approval/rejection signal, do not proceed. Report and stop." Plus an unconditional "never run the gated op in turn 1" — otherwise a sub-agent under auto-mode pressure will rationalize a "trivial case" exception.

**An `idle_notification` is not a report.** A named agent that finishes its work
emits `{"type":"idle_notification","idleReason":"available"}` with no payload —
it says the agent is free, not that it succeeded, and it arrives again on every
subsequent idle. The result only reaches the parent if the agent calls
SendMessage or the parent asks. Treat a bare idle as "ask for the report", and
never as the approval signal for a gate the parent still owes its user.

Seen again in an SDD task-reviewer dispatch (2026-08-11): a review subagent
went idle twice in a row with no content, and a first `SendMessage` asking for
the report also drew only another bare idle — the report only came through on
a second, more explicit ask ("are you finished? if so, output the report text
now"). One "please send it" is not guaranteed to unstick the agent; be ready
to re-ask with a sharper, more direct request before assuming the agent is
actually stuck rather than merely unprompted.

Recurred repeatedly through an SDD run the next day (2026-08-12): three
different subagents (a re-reviewer, and two more task reviewers) each went
idle 2-3 times running with zero chat content, surviving even a sharper
second ask in one case. **Reliable mitigation: tell every dispatched
subagent up front to ALSO write its final report to a named file** (e.g.
"write your report to `<path>`, then reply with just the word 'written'"
or "in case chat delivery is unreliable, also write the same report to
`<path>` as a backup"). Every time this was in the original dispatch
prompt, the file appeared on the next idle even when chat delivery stayed
silent — checking the file directly resolved the stall in one round trip
instead of repeated re-asks. Cheap enough to make standard practice for
any SDD reviewer/re-reviewer dispatch, not just a fallback after the first
stall.

Confirmed Plan 04 Step 3 (2026-05-22) — the old `agents/memory-merger.md` contract had the failed pattern and the sub-agent's own post-mortem identified the gap. Rewritten to two-turn flow; `commands/gitlore/resolve.md` dispatcher updated to match.

Related: [[cc-agent-discovery]] (the namespacing + caching rules that constrain how the contract can be tested in-session).
