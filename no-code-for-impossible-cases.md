---
name: no-code-for-impossible-cases
description: "\"if they exist\" means go check the data model, not build for it"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8e7ea954-eab3-4514-8af6-9e6f60aa7d2e
  modified: 2026-08-10T19:01:13.801Z
---

Before handling a case, establish it can occur. Check the data model and
the upstream source; if they cannot produce the state, do not write code
for it — no guard, no fallback, no test.

The check is usually a cardinality argument. A comment carrying at most
one `parent_id` cannot produce a cycle reachable from a root — every
member of a cycle has its parent inside the cycle — so a walk that only
descends from roots cannot loop, and a `visited` set defends nothing.
GitHub discussions nest exactly one level, a reply cannot be replied to,
so arbitrary-depth recursion with an h6 clamp generalizes over a case the
source cannot emit. In both, the docstring already stated the constraint
the code was written to violate.

**Why:** the cost is real (a NamedTuple to duck the arg-count lint the
guard caused, a second loop, a warning branch, tests) and the benefit is
zero. Worse, a passing test for an
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
