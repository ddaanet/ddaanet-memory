---
name: git-add-untracked-deletion
description: "exit 128 `did not match any files` for a path gone and never in the index; guard with plain `git ls-files`"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7d5af2b9-5693-46c1-9d6b-c3bc01e2a2ec
  modified: 2026-09-20T13:47:48.299Z
---

A path that is gitignored, absent from disk, and never entered the index cannot
be staged at all — not even with `-f`:

    $ git add -f ignored.txt
    fatal: pathspec 'ignored.txt' did not match any files      # exit 128

Verified on git 2.47.3. `-f` overrides the *ignore* rule, not the "no such
pathspec" rule: with the file gone there is nothing on disk to match and no
index entry to record a deletion against, so the pathspec matches nothing.

**Why:** this is the benign failure inside a hook that stages what a manifest
lists, deletions included. Every genuinely tracked path stages fine; the only
128 comes from a removal of a file that was never tracked, where there is also
nothing to stage. A hook that reports staging failures loudly — as it should —
will report this one, and the noise is what tempts the next reader to put a
`2>/dev/null` back on the whole loop (the `shell-scripting:shell-gotchas` skill, `references/robustness.md`), re-blinding
every real failure (a stranded `index.lock`, whose error the `sandbox-lies`
plugin annotates).

**How to apply:** separate the case *before* the `git add`, with plain
`git ls-files -- "$path"`:

- `git ls-files --error-unmatch` answers the question but writes its refusal to
  **stderr**, which for a hook on `exit 0` reaches neither audience (the
  `plugin-craft:hook-authoring` skill) and for a script means a second
  suppression decision.
- Plain `git ls-files -- "$path"` exits **0 either way** and distinguishes by
  *output*: a line for a tracked path, nothing for an untracked one.

So read its exit status explicitly and test the output separately. Do not fold
the two into `[ -z "$(git ls-files -- "$p")" ]`: that spelling discards the
command's status and reads a git failure as "not tracked" (the
`shell-scripting:shell-gotchas` skill).
