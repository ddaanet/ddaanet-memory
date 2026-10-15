---
name: bash32-macos-traps
description: "Apple bash 3.2.57 and macOS ctype traps that pass on Linux bash 5 — temp LC_ALL before a builtin ignored, printf -v %b \"\" keeps the old buffer, failed source exits under set -e despite ||, $p’ glued name; plus how to build 3.2 on Linux"
metadata:
  node_type: memory
  type: reference
  originSessionId: 0c76dad6-3994-43d8-8feb-ca62743586f4
  modified: 2026-10-15T08:02:29.215Z
---

Found 2026-10-14 in sandbox-lies running bats under the Mac's `/bin/bash`
(3.2.57); each confirmed against a 3.2.57 build and, for the first, the
source (`variables.c` `assign_in_env` never calls the locale hook).

- **`LC_ALL=C printf %q …` does not change the locale in 3.2** — a temporary
  assignment before a *builtin* is ignored. External commands (`LC_ALL=C sed`)
  are fine. Use a plain assignment: `$(LC_ALL=C; printf %q "$x")`, or
  save/assign/restore (`saved=${LC_ALL-} had=${LC_ALL+x}` … `unset LC_ALL`
  when `had` is empty). Not `local LC_ALL=C`: 3.2.0 does not undo it.
- **macOS ctype treats bytes 0x80–0xFF as Latin-1 in a UTF-8 locale**, so
  under a UTF-8 locale `printf %q` leaves `é` raw and bash reads `$p’` as a
  variable named `p` plus the quote's first byte (unbound under `set -u`).
  Brace it: `${p}’`. Glibc never shows either, so Linux tests can't catch it.
- **`printf -v x '%b' ""` leaves `x` holding the previous `printf -v`
  buffer** in 3.2. Skip the call when the argument is empty.
- **Under `set -e`, a failed `source` exits 1** in 3.2 even as
  `source f || exit 2`, `if ! source f`, `{ source f; } || …`. Wrap it in
  `set +e` … `rc=$?` … `set -e`.
- macOS awk aborts on an invalid UTF-8 byte in its input (`towc: multibyte
  conversion failure`); APFS refuses a non-UTF-8 file name (`Illegal byte
  sequence`).

**Building 3.2.57 on Debian (gcc 14):** the tarball's `y.tab.c` is stale
against its patched `parse.y`, so delete `y.tab.[ch]` and regenerate it with
byacc (needs only a C compiler; bison needs m4). Then
`./configure --without-bash-malloc --disable-nls` and `make
YACC="<byacc>/yacc -d" CC="gcc -std=gnu89 -fpermissive -w"
CC_FOR_BUILD="<same>"`. To run a bats suite under a given bash, put a `bash`
symlink first on `PATH`: bats and `#!/usr/bin/env bash` scripts all follow it
(sandbox-lies `scripts/run-bats-under.sh`).

Related: the `shell-scripting:shell-gotchas` skill, whose portability
reference is the maintained home for traps like these.
