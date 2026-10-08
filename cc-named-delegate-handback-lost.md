---
name: cc-named-delegate-handback-lost
description: "`Nothing was sent: the agent that spawned you is no longer running.` from `SubagentHandback` while the spawner waits; the spawner's `Agent` result says `ended without delivering a report through SubagentHandback`: the spawner was launched with an `Agent` `name`, making it a teammate: launch a dispatching delegate unnamed; a subagent's `SendMessage` to the delegate's name arrives only once the delegate goes idle; resuming by `SendMessage` sends the `[Subagent hand-back]` to the main session"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9050cbf7-2b4b-4837-b853-fd2b9a6ea330
  modified: 2026-09-28T16:10:45.688Z
---

A delegate launched with the `Agent` tool's `name` parameter is a teammate.
Its spawn result reads "will receive instructions via mailbox". It has no
`SubagentHandback`, and it sends and receives only by `SendMessage`. A
delegate launched without `name` is a plain background agent ("Async agent
launched"). It has `SubagentHandback`, and its subagents' hand-backs come back
as their `Agent` results.

That difference decides how a delegate's own dispatches can report
(auto mode, CC 2.1.281, each row probed):

| Delegate | The subagent reports by | It arrives |
| --- | --- | --- |
| unnamed | `SubagentHandback` | as the `Agent` result, in the same turn |
| named | `SendMessage` to the delegate's name | once the delegate goes idle, resuming it |
| either | a named output file | when the delegate reads it |

Under a named delegate the harness still offers `SubagentHandback` and tells
the subagent that only that call reaches its caller. Every call returns
`Nothing was sent: the agent that spawned you is no longer running.`, while
the delegate is alive and blocked on the call. The delegate's `Agent` result
then reads `The subagent ended without delivering a report through
SubagentHandback` and suggests `SendMessage`. Following that advice sends the
resumed run's `[Subagent hand-back]` to the main session. A subagent's
`SendMessage` to the delegate's name does reach it. It stayed queued through
the delegate's next five tool calls and arrived as a `<teammate-message>` the
moment the delegate went idle, resuming it within the same user turn.

- **A delegate that dispatches is launched without `name`.** Address it by the
  agent id from its spawn result, for gate answers and resumes alike.
- A named delegate's dispatch either writes a file, or is told the delegate's
  name and reports by `SendMessage`. In the second case, the delegate goes
  idle after the dispatch returns, and the report resumes it. An agent type
  whose tool list lacks both `Write` and `SendMessage` (e.g.
  `plugin-dev:skill-reviewer`) has no channel at all.
- Never resume a named delegate's subagent to collect a report.

The documentation says a launcher receives its nested subagents' results and
does not mention the named case. anthropics/claude-code#96225 reports the same
hand-back error for an agent-team teammate's subagent on 2.1.280. Re-probe on a
newer CC version.

Related: [[cc-nested-subagent-spawning]], [[cc-subagent-approval]].
