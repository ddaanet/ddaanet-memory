---
name: cc-bash-background-no-duration-cap
description: "a gate or suite longer than the Bash tool's 10-minute cap: run it with `run_in_background: true` — the cap bounds only the foreground wait, a background task has no duration cap and survives turn boundaries in the main session; the 'terminated at your final response' text is the subagent case"
metadata:
  type: reference
---

The Bash tool's `timeout` (default 120000 ms, max 600000 ms) bounds how long a
command is *awaited in the foreground*. A task launched with
`run_in_background: true` has no duration cap: the timeout is computed and never
applied as a wait, the task is registered, and the tool returns. A foreground
command that hits its timeout is moved to the background when the harness
classifies it as backgroundable, not killed. A background task ends only by
exiting, by a turn abort, or — for a subagent — when the agent gives its final
response, since that ends the subagent's context.

Verified against CC 2.1.261 by decompiling the Bun-embedded bundle
(the gitlore repo's `plans/index-edit-propagation/background-run-timeout-probe.md`),
then operationally: a nine-minute `just precommit` launched in the background
survived the main agent ending its turn, completed, and recorded its sentinel.

**How to apply:** a gate longer than ten minutes is not a reason to split it
into per-recipe calls or to skip recording its pass. Launch it in the
background and read the completion notification; arm a Monitor only when the
result must be acted on inside the same turn. Split calls remain the fallback
when the run dies, and then say the sentinel was not recorded.

Related: [[gate-cache-must-cover-every-check]] for why a recorded pass matters.
