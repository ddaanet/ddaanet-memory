---
name: vibe-bg-process-stops
description: "a long gate raised through the Vibe session's background-process tooling hangs: its job-control worker groups sit SIGSTOP'd (state `T`, wchan `do_signal_stop`) and never resume — SIGCONT does not stick; relaunch detached with `setsid bash -c '… </dev/null'` and poll the log"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8d5debc5-274f-1d8a-16d1-611896bee9f8
  modified: 2026-11-25T17:31:27.000Z
---

A command spanning the session's background-process tooling that spawns its own
job-control groups — measured 2026-11-25 on `just test-unit` driving
`parallel --jobs 4` over bats in gitlore — hangs partway: the worker groups sit
in state `T` with wchan `do_signal_stop`, the gate files never write, and the
log stops growing. Resuming with `SIGCONT` does not stick — the workers return
to `T` within seconds, on terminal I/O or by the spawn harness's own stop
(mechanism unverified; nothing in a log names the stopper, and the stopped
groups are always inside the spawn harness's process tree).

The remedy is to launch detached from that process tree:
`setsid bash -c 'cd <repo> && just <gate> < /dev/null > <log> 2>&1; echo EXIT=$? >> <log>' < /dev/null > /dev/null 2>&1 &`,
then poll the log. A detached run drives to completion without stopping.
Cleaning a stalled attempt first: kill its `parallel`/runner wrapper
`--force`-fully, since orphaned stopped grandchildren survive polite signals.
