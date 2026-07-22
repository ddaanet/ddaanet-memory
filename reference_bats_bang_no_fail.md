---
name: reference_bats_bang_no_fail
description: "a bare `! cmd` inside a bats @test does not fail the test (shellcheck SC2314) — use `run ! cmd`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: e2a4dfcf-7b44-4f1c-bfa8-47c72f899330
  modified: 2026-07-22T05:36:10.407Z
---

In bats, a top-level `! command` line inside a `@test` body does **not** make
the test fail. Bash's `!` inverts the exit status, but bats only inspects the
last command's status, and under `set -e`-like semantics a `!`-negated line is
treated as deliberately-ignored: a non-final one is skipped entirely, and even
as the last line it silently passes on bats < 1.5. So `! grep -qF 'x' file` reads
like "assert x is absent" but asserts nothing. shellcheck flags it **SC2314**
(error when non-final, note when last).

Use `run ! command` (bats ≥ 1.5.0, after `bats_require_minimum_version 1.5.0`),
which runs the command and asserts it failed — the real negative assertion.

Observed 2026-07-21: an implementation plan's test code carried four `! grep -qF`
negative assertions; two would have passed unconditionally, including the one
meant to prove that mirror-down strips the tier prefix from a path. The linter
caught it before the tests ran, not the author's review.

Related: [[reference_shellcheck_comment_directive]], [[feedback_no_handrun_tests]].
