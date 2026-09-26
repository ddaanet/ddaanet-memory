---
name: git-config-multivalued-read
description: "`--get` prints the LAST value and exits 0 on a multi-valued key (`remote.origin.pushurl`), and `--unset` refuses one with status 5, so a guard names one of several values and advises a remedy that errors; use `--get-all`/`--unset-all`. Unset key exits 1; `fatal: bad config line` breaks EVERY git call with 128, not just `git config`. `git config -z --get-regexp \"$key\"` (the shared whitespace default) is a regex over the key name, so an interpolated branch or remote name silently returns another key's value at status 0 (`.`) or misses its own at status 1 (`+`); interpolate into `--get-all` and read one value per line, and keep `-z --get-regexp` for hand-written patterns, whose NUL records survive `read -r -d ''` but not `$( … )`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a3e6885b-b453-42f1-a3f7-16549821ad2f
  modified: 2026-09-26T14:23:44.022Z
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

`-z --get-regexp` is the whitespace-robust enumeration form, and it stays that
for a *literal* pattern. It breaks the moment the key is **interpolated**: the
argument is a regex over the key name, and a branch or remote name legitimately
contains `.`, `+`, `$`, `(`, `)`, `|`, `{`, `}`. Both directions are silent
(measured on git 2.47.3):

- `.` over-matches. `--get-regexp "branch.a.b.merge"` returns
  `branch.aXb.merge`'s value too, status 0 — a wrong value with no sign of it.
- `+` under-matches. `--get-regexp "branch.feat+x.merge"` misses the key of
  exactly that name, status 1, indistinguishable from unset.

So interpolate into `--get-all` with the key as a literal, and reach for
`-z --get-regexp` only with a pattern written by hand. Its NUL records do
survive into a shell — `while IFS= read -r -d '' rec; do …; done < <(git config
-z --get-regexp …)` recovers a value containing spaces — but not through
`$( … )`, which discards NULs.
