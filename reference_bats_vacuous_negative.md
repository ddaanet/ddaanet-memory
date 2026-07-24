---
name: reference-bats-vacuous-negative
description: "`run missing_function` exits 127, so `[ $status -ne 0 ]` passes before the code exists; every predicate needs a positive assertion to have a real RED phase"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 94e0a066-a634-4a25-a32d-b0f53b992c25
  modified: 2026-07-24T17:52:49.992Z
---

A bats assertion of the shape

```bash
run some_predicate <<< "input"
[ "$status" -ne 0 ]     # expecting "false"
```

passes when `some_predicate` **does not exist**: `run` captures the 127
"command not found" status, which satisfies `-ne 0`. Observed 2026-07-20 while
TDD-ing `is_unknown_command` — two of three predicate tests went green during
the RED phase, before a line of implementation was written.

**Why it matters:** it defeats the point of watching the test fail. A negative
assertion proves nothing about a function that isn't there yet, so a predicate
covered *only* by false-cases has no RED phase at all — and would stay green if
the function were later deleted or renamed.

**How to apply:** every predicate needs at least one **positive** assertion
(`[ "$status" -eq 0 ]`) — that is the one that actually pins existence and
behavior. Treat negative cases as refinements around it, not as coverage on
their own. Same trap applies to any `run`-based check keyed on non-zero exit,
including `run !` forms.

Related: [[reference_shellcheck_lints_bats]] (ShellCheck catches the adjacent
bug — a bare `! cmd` mid-body does not fail a bats test at all),
[[reference_bats_errexit]], [[feedback_mutation_check_negatives]] (the general
principle this is a specific instance of).
