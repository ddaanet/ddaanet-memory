---
name: concrete-paths-in-commands-for-the-user
description: "Command blocks I hand my human partner get pasted and run verbatim — use concrete paths, and bind a shared root to an env variable set in the block itself"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: bc8e4078-db3a-4168-8b66-b5c681f7ace1
  modified: 2026-07-21T08:08:37.045Z
---

When I print a command block for my human partner to run — because a
tool call was denied, or because it needs their judgement — they paste
and execute it **verbatim**. Every path must be concrete.

Where several commands share a long root, do not abbreviate it into a
placeholder. Bind it to an env variable **in the first line of the same
block**, so the block stays runnable as-is:

```
SCRATCH=/tmp/claude-1000/-Users-david-code-micro/<session>/scratchpad
cp "$SCRATCH/handoff-task.md" .claude/handoff-task.md
```

What went wrong without this: a `cp <saved>/handoff-task.md …` in the
middle of a 4-command block failed with `bash: saved: No such file or
directory`, and because the surrounding commands succeeded, the
sequence completed looking fine while silently dropping the one file it
existed to preserve.

**Why:** the failure is quiet. `git reset` and `git commit --amend`
either side reported success, so the outcome read as done. Only an
explicit diff against the pre-rewrite tip caught it.

**How to apply:** substitute every real value before printing —
absolute paths, saved SHAs, actual filenames — or bind them as
variables at the top of the block. When the block's purpose is to
preserve something, also state the check that proves it worked, so a
silent skip is visible rather than assumed away.

Related: [[always-write-drafts-to-file]].
