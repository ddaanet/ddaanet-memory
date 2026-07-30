---
name: optional-means-source-can-omit
description: A field is optional only where the upstream source can really omit it; leniency hides a dropped field as a silent null
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 66148f27-8da9-4889-9d90-b64538c0c955
---

Make a field optional (or give it a default) only where the upstream
source can genuinely omit it. Never optional "to be safe", "for
robustness", or to make a fixture easier to write.

**Why:** a defaulted field cannot distinguish *absent because the source
said so* from *absent because we stopped asking for it*. In a decode
chain, the second is a silent, whole-corpus data loss with no error
anywhere — the struct defaults to `None`, the next layer accepts the
`None`, and the column is quietly empty forever. Requiredness is what
turns that into a loud failure at the boundary that caused it. This is
[[no-code-for-impossible-cases]] pointed at schemas: check what the
source actually guarantees rather than hedging.

**How to apply:** for each nullable-looking field, go read the upstream
schema before choosing. Required there → required here. Only genuine
absence earns an optional. Where a layer has a *different* answer from
its neighbour, that asymmetry is real information — write it down rather
than flattening both to optional.

Demonstrated on ghmem: `authorAssociation` was uniformly
`str | None = None` across the wire structs, the ledger contract, and
the columns, though GraphQL types it `CommentAuthorAssociation!` —
non-null everywhere it appears. Dropping it from the query passed every
test and every layer, nulling the column corpus-wide. Making it required
closed it; making it required also *immediately* caught seven fixture
nodes that had already drifted without it. `mergedBy`/`mergedAt` stayed
optional, because an Issue node truly has neither — the distinction the
uniform-optional style had erased.
