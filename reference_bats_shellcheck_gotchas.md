---
name: reference-bats-shellcheck-gotchas
description: "SC2314 bare `! cmd` asserts nothing (`run !`); `run missing_fn` exits 127 (vacuous negatives); `grep -qF` matches any ONE line of a multi-line pattern; @test runs under errexit; shellcheck lints .bats; `# shellcheck` = directive"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-07-29T19:08:13.048Z
---

Independent mechanical facts sharing one trigger surface: writing or reviewing
bats tests and shellchecking them.

Three of them are ways a bats test can look green while testing nothing: a bare
`!` asserts nothing, a `run` of a missing function exits 127 so a negative
assertion passes vacuously, and `grep -qF` over a multi-line pattern asserts
only that some one line of it is present.

The errexit fact points the **opposite** way and exists to refute the
"masked failures" reading — a failing mid-body assertion *aborts* the test
rather than being silently swallowed. Do not file it alongside the other two.

## A bare `! cmd` in a `@test` does not fail the test

A top-level `! command` line inside a `@test` body does **not** make the test
fail. Bash's `!` inverts the exit status, but bats only inspects the last
command's status, and under `set -e`-like semantics a `!`-negated line is
treated as deliberately-ignored: a non-final one is skipped entirely, and even
as the last line it silently passes on bats < 1.5. So `! grep -qF 'x' file`
reads like "assert x is absent" but asserts nothing. ShellCheck flags it
**SC2314** (error when non-final, note when last).

Use `run ! command` (bats ≥ 1.5.0, after `bats_require_minimum_version 1.5.0`),
which runs the command and asserts it failed — the real negative assertion.

Observed 2026-07-21: an implementation plan's test code carried four
`! grep -qF` negative assertions; two would have passed unconditionally,
including the one meant to prove that mirror-down strips the tier prefix from a
path. The linter caught it before the tests ran, not the author's review.
See [[feedback_no_handrun_tests]].

## `run missing_function` exits 127, so negative assertions pass vacuously

```bash
run some_predicate <<< "input"
[ "$status" -ne 0 ]     # expecting "false"
```

passes when `some_predicate` **does not exist**: `run` captures the 127
"command not found" status, which satisfies `-ne 0`. Observed 2026-07-20 while
TDD-ing `is_unknown_command` — two of three predicate tests went green during
the RED phase, before a line of implementation was written.

**Why it matters:** it defeats the point of watching the test fail. A predicate
covered *only* by false-cases has no RED phase at all, and would stay green if
the function were later deleted or renamed.

**How to apply:** every predicate needs at least one **positive** assertion
(`[ "$status" -eq 0 ]`) — that is the one that pins existence and behavior.
Treat negative cases as refinements around it, not as coverage on their own.
Same trap applies to any `run`-based check keyed on non-zero exit, including
`run !` forms. General principle: [[feedback_mutation_check_negatives]].

## `grep -qF "$multiline"` matches any ONE line of the pattern

`grep` is line-oriented: a multi-line `-F` pattern is a *set* of fixed
strings, and matching any one of them satisfies it. So
`echo "$output" | grep -qF "$EXPECTED"` reads as "the whole thing is present"
and asserts only that some single line of it is — it stays green when the
text is reordered, split across the output, or has most of it dropped.
`grep -q` (BRE) is line-oriented the same way and additionally re-reads the
pattern as a regex; `-z` makes grep NUL-delimited rather than multi-line
aware, so it is not the fix either.

Bash substring matching is the assertion that means it:
`[[ "$output" == *"$EXPECTED"* ]]` — contiguous and verbatim, newlines
included. Frame it to assert structure:
`[[ "$output" == *$'\n\n'"$EXPECTED"$'\n\n'* ]]` says the text stands as its
own blank-line-delimited block rather than being spliced into a sentence.

The fixture is half of it: a single-line stand-in for a value that is
multi-line in production makes the two renderings indistinguishable, so
there is no assertion to write. Shape the fixture after the real value.

## `@test` bodies run under errexit

Verified on Bats 1.11.1 (2026-06-08). A failing mid-body `[ … ]`, `[[ … ]]`,
failing pipeline (`echo x | grep -q NOMATCH`), or `jq -e '…' >/dev/null`
evaluating false each **aborts the test** — they are NOT "masked/swallowed".
`>/dev/null` redirects stdout only, never the exit status.

So guarding non-final assertions with `|| return 1` / `|| fail` is a
style/diagnostics choice (bats already prints the failing line + number),
**not** a correctness fix. The genuine errexit exemptions are the usual set:
commands in `if`/`while`/`until` conditions, the left side of `&&`/`||`,
`! cmd`, and non-final elements of a pipeline (without pipefail) — only
assertions hitting those are truly swallowed, and a faithful conversion rarely
produces them.

Context: a code-quality review subagent flagged every non-final assertion in a
converted suite as a correctness HIGH ("masked failures"); a 5-line bats probe
disproved it. See [[feedback-verify-at-source]],
[[project-test-suite-framework-port]].

## ShellCheck lints `.bats` natively

Verified empirically with ShellCheck 0.10.0 on 2026-06-08. It parses
`@test "…" {`, `setup()`, `teardown()`, and `run` with no syntax errors,
descends *into* test bodies (caught a real SC2086 inside an `@test`), and
understands bats' scope model (no false `referenced-but-not-assigned` for vars
shared from `setup()` into a test/teardown, no false positive on `load`ed
`assert_output`). Auto-detected by the `.bats` extension (the 0.10.0 man page
lists `.bash`/`.bats`/`.dash`/`.ksh` plus shell directive/shebang); landed in
ShellCheck 0.7.0 (2019).

So `shellcheck scripts/*.sh tests/*.{sh,bats}` just works — the common
"adopting bats loses `shellcheck -x tests/*.sh` coverage" objection is wrong
for any modern toolchain. Also: `bats` 1.11.1 is already installed at
`/usr/bin/bats`, and the `run`/`$status`/`$output`/`$lines` ergonomics are
bats-core itself — `bats-assert`/`bats-support` are optional sugar, not
required.

## A comment starting `# shellcheck` is parsed as a directive

ShellCheck treats any comment whose first token is `# shellcheck` as a
directive, not prose. A descriptive comment like
`# shellcheck missing on a shell file = ...` triggers SC1072/SC1073
("Expected '=' after directive key") and fails the lint. Reword so the comment
does not begin with the word `shellcheck` (e.g. "Missing shellcheck ...") — it
is otherwise a self-inflicted lint error. Applies to `.bats` files too, since
they are linted.

Related: [[reference-hook-output-channels]] for the exit-2/systemMessage
channels the shell-scripting plugin's shellcheck-on-edit hook uses.
