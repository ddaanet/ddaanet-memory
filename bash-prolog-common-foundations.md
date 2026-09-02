---
name: bash-prolog-common-foundations
description: "what belongs in a shared `bash_prolog` vs in a recipe body"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1da2abcb-5c0f-40ad-ba31-80fe7a6f74f6
  modified: 2026-07-23T09:43:42.193Z
---

A justfile's shared `bash_prolog` is for **common foundations only**: a
function earns its place there by having two or more call sites. Anything
called once gets inlined into the recipe that calls it — including the
tempting `run-<recipe>` wrapper that holds a whole recipe body.

Corollaries my human partner stated in the same breath:

- Early exit is `exit` **in the recipe**, never `return` from a
  `run-<recipe>` helper.
- Setup like `sync` is called once at **recipe toplevel**, not from
  inside a helper that several recipes reach — otherwise a recipe that
  depends on another calls it twice.
- A helper written for cross-project reuse is the one legitimate
  exception to the two-call-site rule: parameterize it (`check-sentinel
  <name>` → `tmp/.<name>-sentinel`) and say so.

**Why:** the prolog is copied between projects, so every function in it
is a thing the next project inherits and has to understand. Single-use
functions there buy indirection and nothing else, and they hide the
recipe's actual shape from someone reading the recipe.

**How to apply:** after editing a prolog, count call sites per function
and inline the singletons. When a recipe grows a helper, ask whether a
second recipe will call it before promoting it out of the body.

Refines the shared justfile shape; the verdict
that recipe prints is the list of checks that passed.
