---
name: reference-bats-errexit
description: "bats `@test` bodies run under errexit; mid-body `[ ]`/`[[ ]]`/failing-pipeline aborts the test, so `|| return 1` guards are style not correctness; verified bats 1.11.1"
metadata: 
  node_type: memory
  type: reference
  originSessionId: d88d17aa-b0e8-4172-ae43-273c924cb8cf
---

Verified on Bats 1.11.1 (2026-06-08): `@test` bodies run with **errexit
active**. A failing mid-body `[ … ]`, `[[ … ]]`, failing pipeline
(`echo x | grep -q NOMATCH`), or `jq -e '…' >/dev/null` evaluating false
each **aborts the test** — they are NOT "masked/swallowed". `>/dev/null`
redirects stdout only, never the exit status.

So guarding non-final assertions with `|| return 1` / `|| fail` is a
style/diagnostics choice (bats already prints the failing line + number),
**not** a correctness fix. The genuine errexit exemptions are the usual
set: commands in `if`/`while`/`until` conditions, the left side of
`&&`/`||`, `! cmd`, and non-final elements of a pipeline (without
pipefail) — only assertions hitting those are truly swallowed, and a
faithful conversion rarely produces them.

Context: a code-quality review subagent flagged every non-final assertion
in the converted suite as a correctness HIGH ("masked failures"); a
5-line bats probe disproved it. See [[reference-shellcheck-lints-bats]],
[[feedback-verify-review-findings]], [[project-test-suite-framework-port]].
