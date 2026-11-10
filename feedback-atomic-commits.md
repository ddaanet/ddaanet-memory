---
name: feedback-atomic-commits
description: "atomic commits per topic, never commit plans/ without being asked, a same-topic follow-up amends, TASK.md and TODO.md amend into the previous commit (handoff, precompact, direct commit) or ride with substantive pending changes"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 80c75686-084a-4258-a87e-356a2210d9d3
  modified: 2026-11-04T00:11:10.528Z
---

Three related rules, same principle (one topic = one commit):

1. **Don't batch unrelated changes.** One topic per commit. Check `git status` before committing and stage only what belongs to the current task. Dotfiles changes, ansible changes, docs changes → separate commits.

2. **Small same-topic follow-ups amend the previous commit, don't create a new one.** A typo fix in a spec just committed, or a missed bullet belonging to the same deliverable, goes in with `git commit --amend`, not a new "fix typo" commit. This contradicts the baseline "prefer new commits" guidance; the atomic-per-topic model takes precedence.

3. **`TASK.md` and `TODO.md` never get a commit of their own.** This holds after a handoff or precompact that commits, and whenever they are committed directly. With nothing substantive pending in `git status`, squash them into the previous commit with `git commit --amend`. With substantive changes pending, they go in the same commit as that new content.

**Why:** My human partner reset a bundled commit once (first rule), pushed back on a standalone follow-up commit asking for an amend instead (second rule), and stated the third rule directly. The corrections point to the same model: a commit is a topic, not a timestamp.

**How to apply:**
- Routine: make new commits.
- Exception: a small tweak to the topic of the *immediately previous* commit (typo, additional bullet in same section, forgotten bit of the same deliverable) is an amend. Confirm with my human partner if unsure.
- When amending is right but would lose a commit someone else has pulled, note that and ask. `just sync` counts: it pushes commits to `dev`, so an amend after a sync diverges the two sides.
