---
name: mutation-sweep-must-survive-a-kill
description: "a `finally` restore does not survive SIGKILL; narrow each mutation's test target and diff the tree afterwards; and a sweep whose writes the sandbox denies (under `.claude/`) reports every mutation SURVIVED — mutate a copy and always run an unmutated control"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 57e023c7-06cb-48c0-afd2-2d4f1906467a
  modified: 2026-07-25T19:29:21.899Z
---

A mutation sweep that mutates a source file, runs pytest, and restores in
a `finally` block is **not** crash-safe. A `finally` runs on exceptions,
not on SIGKILL — and the Bash tool's default 2-minute timeout kills the
runner outright, leaving whichever file was mutated at the moment of death
sitting mutated in the working tree.

**Why:** ghmem's suite takes ~30s, so a five-mutation sweep running the
whole suite each time is guaranteed to exceed the default timeout. It did,
and `triage.py` was left carrying a hardcoded `"diff_valve_dropped": 0` —
which the next `git diff` would have read as intentional work.

**A sweep can also fail the other way: never mutating at all.** The Bash
sandbox denies writes under `.claude/`, so a sweep over a hook script
runs its writes into an error and every mutation reports SURVIVED —
which reads exactly like a suite with no teeth. Mutate a *copy* of the
tree instead, laid out so the test file still finds its subject
(`$TMPDIR/mut/.claude/hooks/…` beside `…/hooks/tests/…`), and always run
one unmutated control through the same harness. A control that
"survives" is the sweep telling you it never ran.

**How to apply:** give each mutation the specific test file that covers it
rather than the whole suite (a targeted run is seconds, and a mutation no
targeted test can kill is the finding anyway), pass an explicit long
`timeout` to the Bash call, assert the mutation target exists before
writing, and **always** `git diff` the mutated paths after the sweep
before trusting the tree. Related:
[[mutate-code-to-test-the-tests]],
[[unkillable-mutation-means-a-wrong-model]],
[[sandbox-char-device-listings]].
