---
name: machine-resource-limits
description: "~2GB RAM and OOM is what kills this box (not load average); /tmp is a 987M tmpfs — run SDD sequentially in the main tree"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8bc34870-8b8c-49fa-85a0-4ad6d99085e8
  modified: 2026-07-21T20:52:17.150Z
---

This development machine will not take parallel heavy work. On 2026-07-21
I ran three SDD implementers concurrently in three git worktrees, each
with its own `uv` venv, each running the full ghmem pytest suite — and
**thrashed the box badly enough that my human partner had to reboot it.**

Two distinct limits, both real:

- **`/tmp` is a 987M tmpfs.** Three worktree venvs (158M each when `uv`
  copies rather than hardlinks) plus three checkouts filled it to 100%.
  Everything then failed with ENOSPC — including the Claude Code
  harness's own stdout capture, so Bash returned "output was lost" and
  the subagents fell silent mid-task with no report. `uv` only hardlinks
  from `~/.cache/uv` when the target shares its filesystem, so a venv
  under `/tmp` is a full copy.
- **RAM is ~2GB total, and OOM is what actually kills this box.** Not
  load average — my human partner's correction: "the load average is
  fine, what kills a system is OOM." A measured snapshot: 1973MB total,
  ~500MB available, 4G swap barely touched. Even after moving the
  worktrees to `/dev/vda1` (19G free), three concurrent pytest suites
  exceeded that envelope. Judge parallelism by **resident memory per
  process against ~500MB of headroom**, never by CPU contention, and
  never treat a high load average as evidence of anything — on a shared
  box it usually means other sessions.
- **Something in the pre-commit hook is a memory hog** — my human
  partner's own diagnosis after the reboot. So a commit is not a cheap
  operation here: never run commits concurrently with a test suite or
  with each other, and treat the commit step as its own quiet moment.

**For big scratch, use `tmp/` inside the project**, not `/tmp` and not
the session scratchpad — my human partner's instruction. It is on the
19G `/dev/vda1` and keeps `uv` hardlinking against `~/.cache/uv`.

**How to apply:** run SDD plans in this repo **sequentially** — one
implementer at a time, in the main tree, against the existing
`tools/ghmem/.venv`. That is the configuration every prior ghmem part
used successfully. Do not parallelize across worktrees here even when a
plan's execution-order section says tasks are independent; independence
is about file conflicts, not about what the hardware can run at once.
If parallelism is ever wanted again, cap it at two and keep worktrees on
real disk, never `/tmp`.

**Other sessions share this envelope.** My human partner often has
several Claude sessions running at once, so the ~500MB of headroom is
not mine alone and can shrink without warning. Budget against what is
free at the moment, not against the total.

**Why:** the failure is silent and expensive. Subagents do not report
ENOSPC or an OOM kill as a blocker — they just stop, and the work looks
abandoned rather than broken. See [[sdd-durable-state-across-compaction]] for the
related lesson about recovering dead subagents from staged work; here
the recovery was `git add -A` + `git diff --cached --binary` into a
patch file per worktree, which preserved everything.
