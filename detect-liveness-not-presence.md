---
name: detect-liveness-not-presence
description: "a foreign tool's state file needs the *current* layout plus an identity marker inside it: layouts move (the old path becomes a declared stray) and abandoned files are never cleaned up, so `[ -f known/path ]` false-positives; fail open, pick newest, stay read-only"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4dd5644b-e0e8-4ec2-8a56-afb8e77f6ff7
  modified: 2026-07-26T13:07:04.133Z
---

When code detects that a *foreign* tool is mid-run by looking for its state
file, `[ -f "$known/path" ]` is not enough. Require the current layout **and** a
positive identity marker inside the file (a header line naming what it belongs
to), and define which one wins when several match.

**Why:** two independent failure modes make presence-at-a-path a false positive.
The layout moves — the other tool relocates its file and often declares the old
path a stray to be ignored, so the one path being tested becomes the one path
that is guaranteed *not* live. And nothing cleans up: a file outside the other
tool's own lifecycle (or left by an abandoned run) survives forever, more so when
it sits in a git-ignored tree where `git status` never shows it. handoff's
`probe_ledger_path` hit both at once on 2026-07-25: a hand-rolled
`.superpowers/sdd/progress.md` left over from a completed plan made a session
running no SDD at all suppress its own `handoff-todo.md` and append findings to
a ledger headed with a different plan's name.

**How to apply:** pin detection to the current layout, read one line and require
the identity marker, and pick most-recently-modified among several matches —
alphabetical glob order is not a liveness signal. Make the near-miss fail *open*
(no detection → do the normal thing), because a false positive here suppresses
your own durable state. Verify the layout and marker against the other tool's
own source, not its docs or your recollection — see
[[verify-at-source]]. Keep the detector read-only: another workflow's
stale file is never yours to delete, however abandoned it looks. And keep one
registry function that owns the layout, with every consumer interpolating what
it prints — a hardcoded copy of the path in a directive drifts silently, and a
glob has no path to hardcode at all.
