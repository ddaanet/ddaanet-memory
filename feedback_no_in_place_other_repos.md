---
name: feedback-no-in-place-other-repos
description: "other repos stay read-only: investigate and propose a patch; the consent boundary is the working tree; a brief dropped there leaves your task frame"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7ffb103e-27dd-4424-9ec0-a51c5fe7d743
  modified: 2026-07-27T15:39:53.283Z
---

When a finding implicates one of David's *other* repos (e.g. `unsandbox-git-status`,
`handoff`, `cwd-safety`), do not edit that repo in place — even when the fix is
small, verified, and the session established the need for it. Investigate, verify,
and *propose*; he applies it himself.

**Why:** the session's working directory is the scope of consent. Another repo has
its own release discipline, its own in-flight work, and its own review path; a
drive-by edit lands uncommitted changes in a tree he did not open. Being right
about the fix does not authorize the edit. Note this is a *broader* rule than
[[reference-sandbox-effects]] (which is about push auth being
classifier-denied): the boundary starts at the working tree, not at the push.

**How to apply:** stay read-only on the *code* of the other tree — no edits to
its sources. Report the diagnosis, the exact change, and the evidence — a patch
or quoted snippet in the reply. David's stated preference (2026-07-17) is that
you may **drop a note in that repo** (e.g. a `NOTES`/`TODO` file describing the
fix) so a session opened in *that* project handles it — the prohibition is on
making the changes yourself, not on leaving a pointer there. Corollary: once he
says "I'll take it from here," stop touching its code; leave what exists rather
than reverting, and say what state it is in.

The note or brief is the *end* of your involvement, not a ticket you hold open.
Do not carry it in a task frame, a handoff, or a pending-work list — a brief
dropped in another repo belongs to a session opened in that repo, and tracking
it here reports work you have no standing to do and cannot observe finishing.
Mention it once when you drop it, then drop it from your own list too.
