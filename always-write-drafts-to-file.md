---
name: always-write-drafts-to-file
description: "Always write drafts (comments, messages, docs, copy) to a file, not just inline in chat"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 17ca47a5-15a1-403f-a87d-643c2fa3b3fd
---

When producing any draft (GitHub comment, email, message, document, marketing copy, etc.),
write it to a file rather than only emitting it inline in the reply.

**Why:** Editing a file across iterations is more efficient than re-outputting the whole
draft each turn, and a file is easier for my human partner to reference and reuse (survives handoff).

**How to apply:** On the first draft, Write it to a sensible path (e.g. `drafts/<slug>.md`)
and point my human partner at it. On revisions, Edit the file instead of pasting the full text again;
in chat, describe just what changed. Keep a short header noting status (e.g. "DRAFT — awaiting
review, do not send"). Related: [[micro-colwrap-project]]
