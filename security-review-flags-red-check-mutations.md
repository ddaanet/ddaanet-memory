---
name: security-review-flags-red-check-mutations
description: "`Background security review found: guard-bypass` / `fail-open` on an uncommitted guard line is often a delegate's red-check mutation in flight; mark each `MUTATION` and gate commits on `grep -rn MUTATION scripts tests`"
metadata:
  node_type: memory
  type: feedback
  originSessionId: ee5f9d2f-0cc7-4f76-8ba3-176a7cc25936
  modified: 2026-10-31T13:58:24.222Z
---

The `security-guidance@claude-code-plugins` background reviewer watches the
working tree as well as commits. While a build delegate does red checks —
disabling one guard condition at a time to prove a test fails — each
mutation arrives in the parent session as a security finding
("fail-open trust-boundary check", "unanchored prefix", "guard-bypass"),
with a suggested fix that is just the original line.

**Why:** across two hook builds in prohibitions, six findings came from
in-flight mutations, each looking like a regression in committed code.
Among them were two real ones: a "commit security review" on committed
code, and a finding on an unmarked committed line. Both named the same
bypass: a substring prefilter ran on raw text, so `git re''set --hard`
slipped past.

**How to apply:**

- Tell every delegate that runs mutation checks to put the word `MUTATION`
  on each mutated line, revert by inverse edit, and run
  `grep -rn MUTATION scripts tests` before each commit, which must be empty.
- Then triage a finding by its source. A marked line in the working tree
  needs no action. A finding on committed code, or on an unmarked line, is
  real until probed. Check with `git grep -n MUTATION HEAD`, then reproduce
  it with one payload through the hook.
- Do not dismiss a commit-review finding just because its details are
  missing. Probe the class it names.

Related: [[guard-safety-visible-in-the-pattern]]; a guard's ALLOW case is
covered by `craft:test-discipline` ("Assert the invocation contract").
