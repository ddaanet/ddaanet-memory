---
name: cc-subagent-bash-cwd-at-spawn
description: "a subagent's Bash calls start in the main session's working directory as it stands when the Agent call is made, so a bare `just release` in a subagent runs wherever the orchestrator's last `cd` left it; `cd` into the project before each dispatch and guard with a bare `pwd`"
metadata:
  node_type: memory
  type: reference
  originSessionId: f50b0e00-9989-4fcf-a0aa-b252b3d1ed5b
  modified: 2026-11-17T19:28:35.876Z
---

A subagent's Bash tool starts each call in the main session's working
directory as it stood at the moment of the `Agent` call. Probed on 2026-11-17
(CC with the Linux bubblewrap sandbox): a Haiku probe spawned right after the
orchestrator had `cd`'d into a sibling repository printed that repository on a
bare `pwd`, and its later calls kept it. Whether the directory is captured at
spawn or read per call was not separated by that probe.

**Why it matters:** the toolkit's `just release` must be a bare call, since a
`cd` prefix keeps it sandboxed and the recipe then refuses in preflight. A
dispatched release agent therefore releases whatever repository the
orchestrator's shell was in, which is the orchestrator's own repository unless
it moved first.

**How to apply:** before dispatching an agent that will run a bare
`just release` or any other bare, cwd-dependent call, `cd` the main session
into that agent's repository and dispatch from there, one agent per `cd`; have
the agent run a bare `pwd` alone in its own call immediately before the bare
call and stop if it is not the expected root. Every other call in the agent
takes an explicit `cd <root> &&` prefix. See [[cc-nested-subagent-spawning]]
for the rest of what a delegate inherits.
