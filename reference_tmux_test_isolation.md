---
name: reference-tmux-test-isolation
description: "TMUX=fake in a test does NOT isolate it from the real tmux server — tmux falls back to the default socket, so a pane id like %0 is somebody's live pane; stub the binary on PATH instead"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 56a001d3-4196-435b-a62f-4dd30ef3dae4
  modified: 2026-07-22T19:02:20.038Z
---

Setting `TMUX=fake` (or any bogus value) in a test env does not stop a `tmux`
call reaching the real server. tmux parses `$TMUX` as `socket,pid,session`, and
on failure falls back to the **default socket** — `/tmp/tmux-$UID/default`. So a
hardcoded pane id like `%0` addresses a live pane belonging to whoever is
running the suite, and `send-keys` types into it.

Verified 2026-07-22 in the handoff plugin: three hooks under test spawn a
detached watcher that drives tmux, and the suite had been firing them at the
real server. It stayed invisible because the watchers died within ~1.5s; a
change that made one poll for 300s hung bats, which is how it surfaced.

**How to apply:** to isolate, stub the `tmux` binary on `PATH` (a shell script
answering `capture-pane` with canned text and swallowing `send-keys`) — the same
thing the watcher unit tests already did. Env-var faking is not isolation for
any tool that has a default connection path; the general form of the question is
"what does this tool do when the env var is unusable?", and for sockets the
answer is usually "connect anyway".

Related: [[reference_cc_tui_tmux_driving]], [[project_precompact_drive]].
