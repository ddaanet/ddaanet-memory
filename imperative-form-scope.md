---
name: imperative-form-scope
description: "the skill-development rule targets instruction subjects (\"You should X\"), not every pronoun; a bare grep for \"you\" over-reports; `your post-compaction self` is a deliberate keep"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f1ddcfd0-b867-4f70-bcd6-76b08a7cf5f3
  modified: 2026-07-23T07:39:07.034Z
---

`plugin-dev:skill-development` requires imperative/infinitive form in skill
bodies. Its prohibited examples are all second person as the **subject of an
instruction** — `You should create...`, `You need to configure...`,
`You can use...`. Reflexive intensifiers (`do not verify it yourself`),
possessives naming a referent (`your post-compaction self`), and passive
objects (`submitted for you`) are not what the rule targets.

**Why:** a bare `grep -w you` over a skill body reports all of these
identically, which inflates a small real fix into a large cosmetic sweep and
invites rewrites that lose meaning. In handoff's two skill bodies, 11 grep
hits contained exactly 1 true violation.

**How to apply:** classify each hit before rewriting — instruction-subject
(rewrite verb-first), load-bearing intensifier (replace the word rather than
delete it: `run it yourself` → `run it directly` keeps the agent-vs-machinery
contrast), removable intensifier (delete), or a referent with no good
third-person label (keep). handoff deliberately keeps `your post-compaction
self` in `skills/precompact/SKILL.md`: it asserts the identity continuity that
motivates writing the file at all, which `the post-compaction agent` drops.
Do not "fix" those two on a later grep sweep. Related:
[[skill-description-purpose-first]].
