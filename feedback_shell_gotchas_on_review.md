---
name: feedback_shell_gotchas_on_review
description: "load `/shell-scripting:shell-gotchas` before *reviewing* .sh/.bats/hook diffs too, not just writing; shellcheck misses the BSD/`set -e`/hook-env class; review unit = the whole touched script"
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
read for the uncatchable one. Related: [[feedback_whitespace_safety]],
[[feedback_no_stderr_suppression]], [[reference_git_hook_env_leak]].
