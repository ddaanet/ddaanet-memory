---
name: feedback-atomic-commits
description: "atomic commits per topic, never commit plans/ without being asked, a same-topic follow-up amends but a review-finding fix is its own commit, TASK.md, TODO.md, a pointer bump or settings noise amend into the previous commit when they belong to it by topic and it is unpushed, else ride the next commit they belong to"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 80c75686-084a-4258-a87e-356a2210d9d3
  modified: 2026-11-17T23:14:03.000Z
---

Three related rules, same principle (one topic = one commit):

1. **Don't batch unrelated changes.** One topic per commit. Check `git status` before committing and stage only what belongs to the current task. Dotfiles changes, ansible changes, docs changes → separate commits.

2. **Small same-topic follow-ups amend the previous commit, don't create a new one.** A typo fix in a spec just committed, or a missed bullet belonging to the same deliverable, goes in with `git commit --amend`, not a new "fix typo" commit. A fix for a review finding is the exception: it is a commit of its own, never an amend, because the per-commit review loop reviews the fix commit. This contradicts the baseline "prefer new commits" guidance; the atomic-per-topic model takes precedence.

3. **Bookkeeping with no content of its own never gets a commit of its own** — `TASK.md`, `TODO.md`, a pointer bump such as the `memory` gitlink, settings noise. This holds after a handoff or precompact that commits, and whenever it is committed directly. When it belongs by topic to the previous commit and that commit is unpushed (`git branch -r --contains HEAD` prints nothing), it is amended into it with `git commit --amend`; otherwise it is left uncommitted to ride the next commit it belongs to, never folded into an unrelated pending topic. A bookkeeping diff that changes behavior is still named in the report (dropped `permissions.allow` entries mean those tools prompt again).

**Why:** My human partner reset a bundled commit once (first rule), pushed back on a standalone follow-up commit asking for an amend instead (second rule), and stated the third rule directly. The corrections point to the same model: a commit is a topic, not a timestamp.

**How to apply:**
- Routine: make new commits.
- Exception: a small tweak to the topic of the *immediately previous* commit (typo, additional bullet in same section, forgotten bit of the same deliverable) is an amend, unless it fixes a review finding. Confirm with my human partner if unsure.
- When amending is right but would lose a commit someone else has pulled, note that and ask. `just sync` counts: it pushes commits to `dev`, so an amend after a sync diverges the two sides.
