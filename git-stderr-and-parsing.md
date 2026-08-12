---
name: git-stderr-and-parsing
description: "which git queries are silent on normal-absence; `(fetch first)`/`(non-fast-forward)` = non-ff, BEHIND *or* diverged (ancestry decides, not wording) vs `(pre-receive hook declined)` = policy; `fetch -q` prints NOTHING on rejection while `push -q` still reports; `--git-common-dir` is relative in a main worktree → CDPATH trap; `-z --get-regexp`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5f90d295-aa18-451c-8ed2-514d78c5207b
  modified: 2026-08-12T08:11:43.804Z
---

Verified empirically 2026-07-20 (git 2.x, Linux) while sweeping `2>/dev/null`
out of gitlore — see [[no-stderr-suppression]].

**Silent on their normal-absence path** (rc≠0, nothing on stderr) — a redirect
on these only ever hides a real fault, so delete it:
`git config --get <missing-key>`, `git config --file <missing-file> <key>`,
`git config --get-regexp` with no match, `git check-ignore -q`,
`git symbolic-ref --short -q HEAD` when detached, `git rev-parse -q --verify
<missing-ref>`, `git rev-parse --show-superproject-working-tree` (rc=0, empty).

**Noisy** — these print `fatal:` and are the genuine "expected message" cases:
`git rev-parse --show-toplevel` and `git symbolic-ref` **outside a repo**;
`awk`/`grep`/`stat` on a missing file.

**Push rejection discriminator** — git's parenthesized reason separates the
classes; the leading marker differs too (`! [rejected]` vs `! [remote rejected]`):
- non-fast-forward → `! [rejected]        HEAD -> live (fetch first)` or
  `(non-fast-forward)`;
- policy → `! [remote rejected] HEAD -> live (pre-receive hook declined)`.
"Updates were rejected" appears only on `hint:` lines and its wording varies, so
it is not a reliable key. Caveat: the parenthesized text is git's UI, not a
documented contract — wording may drift across versions or transports. Pinned
2026-07-21 by `tests/push_rejection_discriminator.bats`, which provokes each
class against a real git (stale-ref push, known-diverged push, local `push .`,
ff-only fetch, `pre-receive` decline) so drift fails there rather than in the
field.

**The parenthesized reason does NOT mean divergence.** git emits the *same*
`(fetch first)` / `(non-fast-forward)` for a ref that is merely **behind** its
target as for one that has genuinely diverged — both are non-fast-forward
pushes, and only the hint prose below differs ("the tip of your current branch
is behind its remote counterpart" vs "the remote contains work that you do not
have"), which is advice text and disableable. Verified 2026-08-12 against a real
remote. So the reason answers "could this fast-forward?", never "is there
something to merge?": ancestry answers that, `merge-base --is-ancestor` in each
direction — pushed⊆target is behind (nothing of ours to publish), target⊆pushed
means the refusal was not about ancestry at all (the target ref is stale or the
remote moved), neither is divergence. Routing a behind ref into a merge ends in
a preparation that finds nothing, reports a merge it could not prepare, and
names a worktree that is **clean** — the reason nobody reads the message. Cost
gitlore a live dead-end in `micro`; a merge preparation that detaches onto the
authority *before* discovering there is nothing to merge also leaves the ref
moved, so a failed diagnosis silently changes the state it was diagnosing.
See [[green-is-not-evidence]] for the shape of the test that catches it: assert
the commit, not a clean `git status`, since the worktree is clean either way.

**`-q` is asymmetric between push and fetch.** `git push -q` still reports its
rejection reason; **`git fetch -q` prints nothing at all on a non-fast-forward**
and only exits 1. So any site that *classifies* a fetch rejection must run
without `-q` — capture stdout+stderr instead. This shipped as a live bug in
gitlore's tier fetch: the divergence arm was unreachable and a diverged tier was
reported as merely "stale". It survived a suite that already covered ff-only
rejection, because that test ran its own `git fetch` **without** `-q` — the
invocation was the bug, not the logic ([[test-the-invocation-path]]).

**Whitespace-safe enumeration:** `git config -z --get-regexp <re>` emits one
NUL-terminated `key\nvalue` record per match, so `${rec#*$'\n'}` recovers a value
containing spaces. Splitting the non-`-z` output on whitespace is broken — for a
submodule *named* with a space it yields part of the key.

**`--git-common-dir` is RELATIVE (`.git`) in a main worktree** and absolute in a
linked one. With an exported CDPATH, `cd "$(git rev-parse --git-common-dir)"`
therefore resolves against CDPATH *and* makes `cd` echo the destination, so the
command substitution captures a two-line, wrong path. Always
`CDPATH='' cd -- "$(…)"`.

**`stat` flavor:** probe once (`stat -c '%Y' .` succeeds on GNU, fails on
BSD/macOS) and memoize, rather than `stat -c … 2>/dev/null || stat -f …` per
file — the per-call form suppresses an expected BSD error every time *and* a
real GNU one.
