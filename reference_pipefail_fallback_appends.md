---
name: reference_pipefail_fallback_appends
description: "`x=$(pipeline || echo FALLBACK)` APPENDS under `pipefail` — a producer failing after the last stage printed captures both; put the `||` outside the substitution, and `return 0` from a function ending in `grep`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6ae1ff29-02e8-4ce4-b1ec-6002914b5c2d
  modified: 2026-07-27T13:57:33.609Z
---

`x=$({ producer; } | filter || echo FALLBACK)` does not mean "use FALLBACK
if the pipeline failed". `pipefail` makes the pipeline fail when **any**
stage fails, including one that failed *after* the last stage already wrote
to stdout — so the substitution captures the real output **and** the
fallback, concatenated.

**Symptom:** `jq: invalid JSON text passed to --argjson`, from a value that
looks like `[]` twice. Any consumer that accepts exactly one document sees
two.

The `||` belongs **outside** the substitution, where it replaces instead of
appending:

```sh
x=$({ producer; } | filter) || x='FALLBACK'
```

The other half of the same incident: a function whose last statement is a
`grep`, a `[ ... ]`, or an `&& printf` returns **1 on its normal
no-match case** — "found nothing" is the expected answer, not an error.
Used as a pipeline producer, `pipefail` reads that as a failed stage. End
such a function with an explicit `return 0`.

Both halves are invisible without `pipefail`, and neither is caught by
`set -e`, which is suspended for a substitution reached from an assignment
with a `||` — see [[reference_errexit_suspended_in_condition]].
