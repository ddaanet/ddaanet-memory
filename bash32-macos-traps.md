---
name: bash32-macos-traps
description: "building Mac's bash 3.2.57 on Debian and running bats under it; the 3.2/macOS traps are in shell-gotchas' portability reference"
metadata:
  node_type: memory
  type: reference
  originSessionId: 0c76dad6-3994-43d8-8feb-ca62743586f4
  modified: 2026-10-30T15:29:53.447Z
---

Mac `/bin/bash` is 3.2.57, and suites green on Linux bash 5 can go red under
it. The traps found that way (temp `LC_ALL` before a builtin ignored,
`printf -v x %b ""` keeping the old buffer, a failed `source` exiting under
`set -e` despite `||`, macOS ctype and `$p’`, awk and APFS on non-UTF-8 bytes)
are maintained in the `shell-scripting:shell-gotchas` skill,
`references/portability.md`. What stays here is how to reproduce them on Linux.

**Building 3.2.57 on Debian (gcc 14):** the tarball's `y.tab.c` is stale
against its patched `parse.y`, so delete `y.tab.[ch]` and regenerate it with
byacc (needs only a C compiler; bison needs m4). Then
`./configure --without-bash-malloc --disable-nls` and `make
YACC="<byacc>/yacc -d" CC="gcc -std=gnu89 -fpermissive -w"
CC_FOR_BUILD="<same>"`.

**Running a bats suite under it:** put a `bash` symlink to the build first on
`PATH`; bats and `#!/usr/bin/env bash` scripts all follow it (sandbox-lies
`scripts/run-bats-under.sh`). macOS ctype traps still need a Mac: glibc never
shows them.
