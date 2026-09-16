---
name: extracting-memory-into-skills
description: "cutting a memory fact into a skill consumers install: a green wikilink/`memory/`-path check still leaves internal ids (`FR5a`, `(D14)`, `NFR1`, `Plan 04 Step 6`) and cross-repo file paths; `my human partner` phrasing is fine and stays; diff date tokens source-vs-output, where a frontmatter `modified:` stamp is the only one that should vanish"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c2b17eb2-0e56-457d-9dad-3cad8729bf96
  modified: 2026-09-16T18:03:44.298Z
---

Moving a memory fact into a skill that ships to consumers is mostly
mechanical, and a `[[wikilink]]` / `memory/`-path check catches most of it.
Two classes survive that check, and each dangles for every consumer:

- **Internal requirement, decision and plan ids** — `FR5a/FR5c`, `(D14)`,
  `against NFR1/D7`, `discovered closing Plan 04 Step 6`. They read as
  precision and resolve to nothing outside the repo whose spec defines them.
  Drop the id and keep the date and the observation: the date grounds the
  claim on its own.
- **File paths in another repository** — a skill body, a test helper, a
  design doc, a `~/code/<repo>` path. Keep the rule and the measurement,
  generalize the illustration. Naming the *project* a finding came from is
  fine and is still evidence; naming a file the reader cannot open is not.

House phrasing for the user — `my human partner` — is **not** in this set.
It ships fine in skill bodies, on the `superpowers` precedent, and my human
partner likes it there. It is policy, not tolerance: "the user" is whoever uses
the software being built, so the collaborator needs a name of its own. A gate
that flags the phrase in a shipped body enforces the opposite of the policy. Rewrite such a line only where it is separately an
imperative-form violation, and then for that reason.

**Why:** the enforcement script sees wikilinks, `memory/` paths and bare fact
filenames, so a green check reads as "no dangling references" while the two
classes above are untouched. Evidence: in the plugin-craft seed extraction the
script was green and an Opus review returned two unresolvable internal ids as
its only critical findings.

**How to apply:** run the mechanical check, then grep separately for
`\b(FR[0-9]+[a-z]?|NFR[0-9]+|D[0-9]{1,2})\b`, for `/Users/` and `~/code/`, and
for source-file extensions outside the shipping repo's own tree. Route the
extracted files through a reviewer one rung above the extractor: fidelity of
*dates and figures* is what to verify mechanically — diff the date tokens
between source and output, where a frontmatter `modified:` stamp is the only
one that should disappear — and reachability of *identifiers* is what needs a
reader.

The underlying rule is that distributed text never cites a private file — a
skill body ships to consumers who do not have the author's notes, so inline the
criteria at write time (`craft:directive-writing`). This extends it past what a
grep sees.
