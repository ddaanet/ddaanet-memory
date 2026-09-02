---
name: no-stderr-suppression
description: "`2>/dev/null` in a script or a diff is a finding to justify, not a neutral idiom: a guard, then capture-and-match, then a justified redirect"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5f90d295-aa18-451c-8ed2-514d78c5207b
  modified: 2026-08-11T18:10:16.135Z
---

My human partner: "Remove `2>/dev/null` everywhere, unless failure is normal
and error message expected... and even there, it would be safer to surface
the error message unless it matches expectation."

**Why:** a blanket redirect silences one anticipated message and every
unanticipated one with it. Two real defects hid behind exactly that: both
`git push` sites assumed failure meant unreachable-or-divergence, so a
protected branch or a pre-receive decline started a merge resolution that could
not help — with git's own explanation discarded; and a suppressed `ls -A` let an
unreadable directory read as "empty", so the installer proceeded into it.

**How to apply:** treat an existing `2>/dev/null`, in a script or in a diff
under review, as a finding that has to be justified rather than a neutral idiom.
Prefer, in order: guard the expected failure away so no redirect is needed and
what remains on stderr is real; capture and match when the command has several
failure modes you treat differently; keep the redirect only where provoking the
error *is* the mechanism, and write that reason inline.
