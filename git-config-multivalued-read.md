---
name: git-config-multivalued-read
description: "`--get` prints the LAST value and exits 0 on a multi-valued key (`remote.origin.pushurl`), and `--unset` refuses one with status 5, so a guard names one of several values and advises a remedy that errors; use `--get-all`/`--unset-all`. Unset key exits 1; `fatal: bad config line` breaks EVERY git call with 128, not just `git config`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a3e6885b-b453-42f1-a3f7-16549821ad2f
  modified: 2026-09-17T10:24:39.266Z
---

Measured on git 2.47.3.

- **`git config --get <key>` on a multi-valued key prints the LAST value and
  exits 0.** It does not fail, and it gives no sign that other values exist.
- **`git config --unset <key>` on a multi-valued key refuses, status 5.**
- **`git config --get <key>` on an unset key exits 1** — the status a guard
  absorbs. `--get-all` keeps that behaviour.
- **A config file git cannot parse breaks EVERY git call with status 128**,
  not just `git config`. So there is no fixture that makes `git config` fail
  with some other status while leaving the rest of a script workable: the
  "absorbs status 1 only vs absorbs everything" distinction is unpinnable by
  a test and has to be a code-review check.

`remote.<name>.pushurl` is genuinely multi-valued — pushing to several
mirrors is a supported arrangement — so this is a real configuration, not a
corner case.

**Why:** a guard that refuses a redirected push route read `--get`, named one
of two push URLs, and advised `unset it`. The refusal was correct and the
remedy errored — worse than silence, because the operator follows it and
gets rc=5 with no explanation. The failure is invisible to review by reading:
`--get` looks like it returns "the value".

**How to apply:** read with `--get-all`, list the values one per line in a
`hint:` block rather than interpolating them into a single-line message (a
path holding a space must not read as two), and advise `--unset-all`.
State the residual in a comment: a value containing a newline still prints
across two lines, and `git config` has no quoting mode that avoids it.

Do **not** reach for `-z --get-regexp` as the whitespace-robust form when the
key is interpolated: its argument is a *regex over the key name*, and a branch
name can legitimately contain `.`, `+`, `$`, `(`, `)`, `|`, `{`, `}`. NUL
delimiting is unobtainable anyway — bash discards NULs in `$( … )`.

See [[errexit-status-propagation]] for the capture rule these reads obey.
