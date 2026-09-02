---
name: design-doc-rewire-dead-components
description: "a doc, spec or decision cites a script, command or subsystem that no longer exists: substitute the successor, generalize where only the principle survived, keep the dead name where the decision is *about* it, or state inline that nothing replaced it — never a stale pointer, never a dropped decision"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a0cb79ca-c935-4c91-8ed2-3e3a5526d159
  modified: 2026-09-02T12:57:13.626Z
---

When a component a living design doc references is removed, **rewire the
document** in the same sweep as the code. Neither of the two easy moves is
right: leaving the stale reference standing, or deleting the decision along
with the thing it named.

**Why:** these documents carry institutional memory, and a decision pointing at
something that no longer exists degrades into noise a future session can neither
act on nor trust. But the decision usually outlives its mechanism — deleting it
invites re-litigating a question already settled for good reason, which is the
rejected-alternatives section's whole job. The audit trail lives in git history;
the document is current truth.

**How to apply** — four rewires, by what survived:

- **A successor exists** → substitute it. This is [[design-doc-writing]]'s
  mechanism-vs-rationale split applied to a name.
- **The mechanism died, the principle survived** → generalize the prose rather
  than delete the decision. "`edify _recall resolve` is the canonical gate
  anchor" becomes "the recall pass is the canonical gate anchor" — same
  decision, one abstraction level up, no dangling name.
- **The decision is *about* the dead component** → keep the name, because
  naming it is the point. Evidence that a precommit gate broke for nine days on
  a non-existent `edify validate` command is worthless with the command
  redacted.
- **Nothing replaced it** → say so inline. A sentence stating the script was
  not restored is a fact a reader can act on; a path that silently resolves to
  nothing is not.

The failure this prevents is asymmetric. A stale pointer is usually caught by
the next reader who follows it; a dropped decision is silent, and its cost
arrives later as someone re-deriving an answer the project already had.

Related: [[design-doc-writing]] for structure and the write-time-record split;
[[design-doc-no-situational-state]] for the entries that should be retired
outright rather than rewired.
