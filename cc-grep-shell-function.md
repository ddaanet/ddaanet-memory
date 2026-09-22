---
name: cc-grep-shell-function
description: "`grep` in the Bash tool is a shell function running `claude` as ugrep (`exec -a ugrep`, `--ignore-files -I`), so a hand probe measures ugrep, not the `/usr/bin/grep` a script gets — probe with `command grep`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 78c01ada-e3fc-450c-b044-3f914c0034dc
  modified: 2026-09-22T08:31:28.510Z
---

Claude Code's Bash tool defines `grep` as a shell function (`type grep` shows
it). Unless an argument matches one of ugrep's own flags (`--filter`,
`--pager`, `-Z`, `--null`, …), it runs the `claude` binary itself with
`exec -a ugrep` and the options `-G --ignore-files --hidden -I
--exclude-dir=.git --exclude-dir=.svn …`, and only falls back to
`command grep` when that binary is missing. Seen on CC 2.1.278.

A script invoked by that shell is not affected — functions are not exported
to child processes — so `tests/*.sh` and hook scripts get the real `grep` on
`PATH`. What is affected is any probe typed directly into the Bash tool to
establish how grep behaves: exit status under `-q` on a closed pipe, whether
`--` is honoured, output on binary files (`-I` is on), whether `.gitignore`d
files are searched (`--ignore-files` is on). One such probe produced a wrong
EPIPE threshold that was carried into a test before the mismatch was caught.

**How to apply:** probe with `command grep` or `/usr/bin/grep`, or run the
probe inside a script file, when the result is going to stand for what a
script sees. Treat a grep result that disagrees with a script's behaviour as
this mismatch before anything else.
