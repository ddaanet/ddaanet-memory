---
name: gate-verdict-recaps-what-passed
description: "A quality gate's success line recaps what passed uneventfully — never a bare OK/Green, which asserts what the silence should be showing"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1da2abcb-5c0f-40ad-ba31-80fe7a6f74f6
  modified: 2026-07-23T09:43:50.589Z
---

When every check in a gate is quiet on success, the sign-off must be a
**one-line recap of what passed uneventfully**, accumulated from the
checks as they run:

```
✓ ruff check · docformatter -c · mypy · 186 passed · line limits
```

Not `✓ Green OK`. Not `✓ All checks passed`. My human partner rejected
both: a bare verdict asserts the very thing the reader is trying to
confirm.

**Why:** the reason a gate needs a verdict at all is that silence is
ambiguous — a check that printed nothing is indistinguishable from a
check that never ran (mypy with `error_summary = false` is the standing
example). A word like "Green" restates the ambiguity one level up. A
recap resolves it, because a check missing from the list is visibly
missing.

**How to apply:** have the reporting helper append its label on exit 0,
and print the accumulated list as the success line. Failure stays terse
(`✗ Red!`) — the failing check has already printed its own section.

Applies to the prolog structure in
[[bash-prolog-common-foundations]].
