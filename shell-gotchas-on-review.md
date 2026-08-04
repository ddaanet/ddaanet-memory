---
name: shell-gotchas-on-review
description: "reviewing `.sh`/`.bats`/hook diffs, not only writing them: load `/shell-scripting:shell-gotchas` first, since shellcheck can't see the BSD/`set -e`/hook-env class"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3010dc03-b7fc-460c-afa5-9c4eb1dfd0d7
  modified: 2026-07-23T13:26:32.789Z
---

Load `/shell-scripting:shell-gotchas` **before** reviewing shell code — a diff
of `.sh`/`.bash`/`.bats`/git-hook/Makefile-recipe content, not only when
authoring it.

**Why:** the skill's value is the catalog ShellCheck cannot see (BSD/macOS
divergence, `set -e` status loss, git-hook env leakage, bash 3.2 limits). A
review that leans on shellcheck plus reading comprehension misses exactly that
class, and its rule "audit the whole touched script, not the diff" changes the
unit of review.

**How to apply:** invoke the skill in the same turn the review is requested,
before opening the first file; then run shellcheck for the mechanical layer and
read for the uncatchable one. Related: [[no-stderr-suppression]],
[[git-hook-env-leak]].
