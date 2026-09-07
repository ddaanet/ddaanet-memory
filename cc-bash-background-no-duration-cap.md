---
name: cc-bash-background-no-duration-cap
description: "a gate longer than the Bash tool's 10-minute cap: run it with `run_in_background: true`; the cap bounds only the foreground wait and the task survives turn boundaries; in a subagent the completion notification may never arrive though the run is fine — read the gate's own pass artifact, never `ps` or buffered stdout; \"terminated at your final response\" is the subagent case"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a9bf3102-9e37-4a28-86a7-939d19b02f31
  modified: 2026-09-07T14:34:10.426Z
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

**The completion notification is not reliable inside a subagent.** Two
test-driver dispatches in one orchestrate run each launched the gate in the
background and then sat idle waiting for a notification that never arrived,
while the run itself completed normally. Neither had died.

Nothing a waiting subagent can observe settles it. `ps` in a fresh Bash call
cannot see another background task's processes at all — each call gets its own
process-tree view, so "no `just` running" is not evidence of death. Stdout does
not settle it either wherever the runner buffers into a log file and prints only
at the end, which is the normal shape for a test wrapper.

**Read the verdict from whatever durable artifact the gate writes on a pass** —
a sentinel file, a cache marker, a report. It is valid for the tree as it stands
when it postdates the last edit to any input the gate covers, and a report
saying the verdict came from there rather than from a notification is honest.
The same artifact recovers a partial run: a killed multi-recipe gate keeps
whatever finished, so only the missing recipes need re-running. A gate with no
such artifact gives a waiting agent nothing, which is an argument for adding
one.

**How to apply:** a gate longer than ten minutes is not a reason to split it
into per-recipe calls or to skip recording its pass. Launch it in the
background; arm a Monitor only when the result must be acted on inside the same
turn. Do not wait on the notification alone. Split calls remain the fallback
when the run genuinely dies, and then say the pass was not recorded.

Related: [[gate-cache-must-cover-every-check]] for why a recorded pass matters.
