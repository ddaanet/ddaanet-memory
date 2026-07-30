---
name: no-doc-history-references
description: Specs/docs must not reference their own previous versions; rejections of real design alternatives stay
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e576f6e1-e871-43d6-9cd2-41c9acd7f36c
  modified: 2026-07-30T15:34:29.790Z
---

When revising a spec or design doc, scrub language of the form "don't
do X (like a previous version of this doc did)" — e.g. "was
confusingly similar", "the earlier constraint is dropped", "the
smoking gun", "(good) not (bad)" comparisons against a prior draft.

**Why:** implementers and future readers don't know and don't care
about previous versions of the document; self-referential history is
noise to them.

**How to apply:** state the surviving design positively. KEEP
rejections of real design alternatives that make sense on their own
("shelling out to `gh api graphql` was rejected: ...", "deny-patterns
were rejected") — those are decisions, not draft archaeology. Also
introduce private/opaque references (sibling repos, personal tooling
like edify) on first use instead of assuming the reader knows them.
