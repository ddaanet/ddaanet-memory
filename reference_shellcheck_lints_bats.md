---
name: reference-shellcheck-lints-bats
description: "ShellCheck lints .bats files natively (verified 0.10.0) — the \"lose lint coverage\" objection to bats is false"
metadata: 
  node_type: memory
  type: reference
  originSessionId: d88d17aa-b0e8-4172-ae43-273c924cb8cf
---

ShellCheck lints `.bats` files natively — verified empirically with
ShellCheck 0.10.0 on 2026-06-08. It parses `@test "…" {`, `setup()`,
`teardown()`, and `run` with no syntax errors, descends *into* test
bodies (caught a real SC2086 inside an `@test`), and understands bats'
scope model (no false `referenced-but-not-assigned` for vars shared
from `setup()` into a test/teardown, no false positive on `load`ed
`assert_output`). Auto-detected by the `.bats` extension. Landed in
ShellCheck 0.7.0.

So `shellcheck scripts/*.sh tests/*.{sh,bats}` just works — the common
"adopting bats loses `shellcheck -x tests/*.sh` coverage" objection is
wrong for any modern toolchain. Also note: `bats` 1.11.1 is already
installed at `/usr/bin/bats`, and the `run`/`$status`/`$output`/`$lines`
ergonomics are bats-core itself — `bats-assert`/`bats-support` are
optional sugar, not required. Relevant to [[project-test-suite-framework-port]].
