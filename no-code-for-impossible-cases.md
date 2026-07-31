---
name: no-code-for-impossible-cases
description: "Don't write defensive code for states the data model or upstream source cannot produce; check the constraint before generalizing"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8e7ea954-eab3-4514-8af6-9e6f60aa7d2e
---

Before handling a case, establish it can occur. Check the data model and
the upstream source; if they cannot produce the state, do not write code
for it — no guard, no fallback, no test.

Two corrections in one session on the ghmem discussion renderer:

- Added a `visited` set to make a comment walk "cycle-safe". A comment
  has at most one `parent_id`, so a cycle's members all have their
  parent inside the cycle, making it unreachable from any root — the
  walk only descends from roots, so it cannot loop. The guard defended
  nothing, and dragged in a NamedTuple purely to duck the arg-count lint
  it caused.
- Generalized that walk to arbitrary depth with an h6 clamp. GitHub
  discussions nest exactly one level: a reply cannot be replied to. The
  existing docstring said so and I wrote a replacement that said so
  while building the general case anyway.

**Why:** the cost is real (a NamedTuple, a second loop, a warning
branch, tests) and the benefit is zero. Worse, a passing test for an
unreachable state reads as verification — it only proves the code
handles input the system cannot produce, so it makes dead machinery
look load-bearing and freezes it against later simplification.

**How to apply:** when my human partner gates a request on a condition
("... *if they exist*", "if that can happen"), that is an instruction to
go check, not a hypothetical to build for. Report what the check found.
Contrast with a case that genuinely occurs — an orphan reply whose
parent is missing from the corpus is real, because an incremental or
paginated fetch can land a reply before its parent; that one earns its
handling. See [[dont-bake-in-guarantees]] for the sibling failure of
over-committing on behalf of hypothetical callers.
