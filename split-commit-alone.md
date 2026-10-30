---
name: split-commit-alone
description: "splitting a file (a doc over its line cap, a module): the split lands in its own commit carrying only the move and the inbound citation updates it forces — never mixed with a reflow, an edit or any other change"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 6a6038a1-c085-481e-bf8b-9ca39c4ef491
  modified: 2026-10-30T16:51:29.613Z
---

A split — moving a section of a file into a new file, for a line cap or a
seam — lands in a clean commit of its own. That commit carries the move and
the inbound references the move forces (links, section citations, a design
doc naming where something lives), and nothing else: no reflow, no rewording,
no content change riding along.

**Why:** my human partner's rule, stated as general while settling the
reflow-and-size-gate plan in shell-gotchas: "splits should not be mixed with
other changes, clean." A pure move reviews as a move; an edit hidden inside
one is invisible in a diff that shows hundreds of moved lines.

**How to apply:** when a plan or a change needs a split, sequence it as its
own step and commit, before or after the edit that motivated it, never
inside it. It is the inverse case of [[commit-bundling]], which keeps
related changes together: a split is the one change that travels alone.
