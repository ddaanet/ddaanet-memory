---
name: code-locality-ordering
description: "Order code top-to-bottom entry-first; define constants and helpers AFTER their users, never front-loaded"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 66148f27-8da9-4889-9d90-b64538c0c955
---

Group code by purpose, not by type, and order it top to bottom: entry
points first, then their dependencies, so a name is known by its purpose
before its definition. Definitions go **after** the code that uses them —
including literals and constants. Do not front-load a constant above the
function that reads it, and do not group all constants (or all classes,
or all queries) into a section of their own. Constants with few uses live
next to their user; a constant used once is usually better inlined.

**Why:** a reader arrives wanting to know what the module *does*, and
meets its entry point first; every definition below is then something
they already have a reason to care about. Front-loading definitions is
C-header habit — it forces the reader to hold N unexplained names in
their head before reaching the one function that gives them meaning.
Python's late binding makes the C order unnecessary, so it buys nothing.

**How to apply:** a name referenced *inside a function body* is late-bound
and can be defined after that function — this is the common case, so
prefer it. The exceptions are the genuine evaluation-order constraints,
where the name is consumed at import or class-creation time:

- a base class must precede its subclasses;
- a constant *composed into another module-level constant* (a query
  fragment substituted into a query, say) must precede it — `substitute`
  runs at import, unlike a body reference;
- msgspec/dataclass field annotations resolve at class creation, so a
  struct's field types must already exist.

When one of these forces a definition upward, place it as late as
evaluation allows — after the first function that needs it, not at the
top of the section — and say so in a docstring, or the next reader will
"fix" the order into a `NameError`.

Confirmed twice on ghmem's `sync.py`: first as the rule ([[ghmem-project]]
cleanup wave 1), then as a correction when I put the GraphQL literals
*above* their functions and was told to invert it.
