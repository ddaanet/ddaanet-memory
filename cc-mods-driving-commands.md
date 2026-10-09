---
name: cc-mods-driving-commands
description: "a mod running `/clear`/`/compact`/`/rename` plus a continuation: `called from a classic.Stop hook … run it from a later event (turn.complete)` — `command.run` rejects from `classic.Stop` even via `$.clock.after`, timers die under `-p` — drive inline in `on('turn.complete')` with cwd recorded at Stop; SessionStart(clear|compact) still fires; `$` only to top-level functions"
metadata:
  node_type: memory
  type: reference
  originSessionId: 379ebca6-dad5-4ac4-a215-3392d6d21885
  modified: 2026-10-09T14:52:59.709Z
---

Verified live on CC 2.1.295 under `claude -p` (see [[cc-mods-api]] for the API
itself).

- **`$.clock.after` does not escape the "inside a hook the turn is waiting on"
  check.** A timer inherits the `classic.Stop` call context:
  `called from a classic.Stop hook … run it from a later event
  (turn.complete)`, for `after(0)` and `after(300)` alike. Under `-p` a timer
  also dies with the session (~2s after `Stop`): `no session is bound`.
- **Drive inline in `on('turn.complete')`.** Awaiting `command.run` and
  `prompt.submit` there does not deadlock and keeps `-p` alive until the
  continuation turn ends (13s clear, 23s compact). The payload has no `cwd`:
  record `e.cwd` in `classic.Stop`, which also keeps Esc (no `Stop`) from
  arming anything.
- **A mod-run `/clear` or `/compact` fires `SessionStart`** (`source: clear` /
  `compact`) about 0.3s after the command, new session id on clear, so classic
  loaders still inject the frame. `prompt.submit({asUser: true})` then lands as
  a plain user prompt in the fresh conversation.
- `command.run` takes the name without a slash and `args` as a string;
  `compact <focus>` resolves only when compaction ends (~7s); `rename` writes a
  last-wins `custom-title` entry.
- A hooks module may pass `$` only to top-level function declarations; an
  inner `const f = ($) => …` fails the load check ("hooks module did not
  load").
