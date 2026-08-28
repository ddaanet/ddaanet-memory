---
name: commit-bundling
description: "which commit a change rides, and what never gets carved out of one silently: related working-tree changes stay in; a documenting note goes with the change it documents, never with unrelated pending work; bare bookkeeping (a pointer bump, settings.json) folds into whatever is convenient, incl. an amend"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5c5b043b-8460-4841-a796-694a07a15915
  modified: 2026-08-14T08:01:07.170Z
---

Bundle what corresponds to the change, and never carve anything out of a
commit silently.

**Related working-tree changes stay in.** Files judged "unrelated" do not get
dropped from a commit without saying so. Where something genuinely should not
be committed, name it and let my human partner decide rather than quietly
leaving it behind. Splitting a `.claude/settings.json` plugin-enablement
change out of a feature commit drew the correction "why you always want to not
commit stuff????" — silent exclusion only makes them notice the gap and
re-ask, where one commit capturing the session's work in one pass was wanted.

A memory-related diff then takes opposite placement rules depending on whether
it carries content of its own.

**A memory note that documents something** — a fix, a decision, a discovered
mechanism — rides with the corresponding source change: one parent-repo
commit carrying both the source diff and the memory submodule's gitlink
bump, not two. Do not bundle it with *unrelated* pending memory work either
— a gitlink bump left over from an earlier, separate task does not belong in
a later fix's commit. Bundle what corresponds, not everything pending.

**Bare bookkeeping with nothing behind it** — the memory submodule pointer
moving with no substantive note attached, `.claude/settings.json` — does not
need a commit of its own or a matching subject, and does not need to wait
for something to correspond to. Fold it into whatever commit is most
convenient, including amending an unrelated commit that just landed; don't
ask which commit it belongs in and don't split it out.

**Why:** After a bug fix in `handoff`'s compaction watcher, the source fix
and the memory submodule's gitlink bump (from unrelated earlier work) were
proposed as two separate commits, with the split offered as a choice. My
human partner rejected the premise: a memory update belongs with the source
change it corresponds to, as a standing rule, not a per-commit judgment
call. Separately, session bookkeeping files got the opposite
correction: "just amend the settings and memory changes to whatever was
committed last" — these are session bookkeeping, not project history, and
keeping them dirty to preserve a tidy commit boundary costs more attention
than the tidiness is worth. A dirty tree also blocks tools that gate on it —
`just release` aborts on `git diff --quiet HEAD`, and its dry-run
`git reset --hard` makes uncommitted work a hazard. The carve-out correction
is the same instinct caught from the other side: the placement rules say
which commit a diff rides, never that a diff may vanish from the set.

**How to apply:** stage the full set of changes by default, and state any
exclusion out loud instead of acting on it. For the memory delta specifically,
ask whether it carries its own substantive content. If yes, stage and commit
it with the source change it documents — never split, never bundled with
unrelated pending memory work. If no — a bare pointer bump or `settings.json`
noise with nothing to say for itself — don't ask which commit it belongs in;
fold it into the current or last commit, including amending one that just
landed, and say so in the report. Either way, still name what the diff
actually does when it changes behavior (e.g. dropping `permissions.allow`
entries means those tools start prompting again) — a project's own
observable-state-reporting rule still applies. Committing the memory pointer
still goes through the gitlore gate: summarize, write
`$(git -C memory rev-parse --git-path gitlore-commit-msg)`, then commit the
parent — never commit inside the submodule.
