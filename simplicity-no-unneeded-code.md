---
name: simplicity-no-unneeded-code
description: "before adding an existence probe, a guard branch or a dedup check, ask whether the unguarded form is already harmless; a PATH entry for a missing directory is inert, so it needs no `-d` test"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 751463d2-2bff-4bb6-8d21-adf62316e635
  modified: 2026-10-26T18:17:54.050Z
---

Simplicity is the governing principle: write no code that is not needed.
My human partner quoted Saint-Exupéry: perfection is reached not when there is
nothing left to add, but when there is nothing left to remove.

The case that prompted it: the dogfood shim (`toolkit/bin/claude`) appended
`<root>/dist/plugin/bin` to PATH only when a copy existed, behind a probe and a
branch. A PATH entry naming a directory that does not exist is inert, so the
probe and the branch bought nothing. Worse, they cost something: a session
launched before the first `just dogfood` never got the entry, and had to be
restarted from the terminal to pick it up.

**Why:** every guard is code to read, test and keep in step with the docs, and
a guard against a harmless state usually also blocks a useful one, here the
first sync into a session that is already running.

**How to apply:** before writing an existence check, a "only when" branch or a
dedup pass, state what goes wrong without it. If the answer is "nothing, the
unguarded form is inert", drop it. When reviewing a proposal or a subagent's
diff, look for guards of this kind and propose removing them. Defending a
guard with a consistency argument ("no release produces that state") is not
enough on its own when the guard also costs a restart or a branch. Related: the
`craft:removing-cleanly` skill.
