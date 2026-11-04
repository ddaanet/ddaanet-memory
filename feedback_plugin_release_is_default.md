---
name: plugin-release-is-default
description: "For ddaanet plugin repos, when local commits are release-worthy, the default \"what's next\" is the full release cascade, not just `git push`"
metadata:
  type: feedback
  originSessionId: f1e8f285-ade0-4db8-9054-af47c3d0bc93
  modified: 2026-11-04T00:25:46.729Z
---

When my human partner asks "next?" on a ddaanet plugin repo (gitmoji, handoff,
skills, edify) with release-worthy commits on `dev` not yet on `main`,
default to the **full release cascade** (the release review of `dev`
against `main`, `main` fast-forwarded to `dev` and `/ddaa:preflight`, then
`just release patch` → marketplace bump → push), not just `git push`. The
toolkit's recipe takes a bump and no flag: `--yes` fails with `Justfile does
not contain recipe`.

**Why:** A bare push to a plugin repo doesn't reach users — installs
resolve through `claude-plugins/.claude-plugin/marketplace.json`, so
shipping requires the tag + release + marketplace bump cascade.
My human partner confirmed: "I meant both" after I offered push-or-release as
alternatives. The two aren't peers; release subsumes push.

**How to apply:** On these repos, if `git log main..dev`
shows commits suitable for a patch release (features, fixes,
refactors, tests — not just WIP), propose the full
release-and-marketplace flow as the primary path. Offer "push only"
only when the commits are deliberately not release-worthy (e.g.,
docs-only, in-flight work).
