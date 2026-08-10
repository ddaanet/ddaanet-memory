---
name: jsonl-reader-type-guard
description: "reading a JSONL transcript line by line: parse success is not object-ness, and the next field access on a scalar is fatal in both jq and Python"
metadata: 
  node_type: memory
  type: reference
  originSessionId: d076e027-4399-4b8e-bc50-6356bb9f4786
  modified: 2026-08-10T07:01:59.654Z
---

A line-by-line JSONL reader needs two guards, not one. Filtering lines that
fail to parse is the obvious half; a line that parses to a **scalar or an
array** passes that filter and kills the reader at the first field access.

**jq.** `fromjson? // empty` filters parse failures (and `null`/`false`). It
does not filter `42`, `"a string"` or `[1,2]`, and indexing one of those is a
fatal error, not a null:

```
$ printf '42\n' | jq -Rn '[inputs | fromjson? // empty | .foo]'
jq: error (at <stdin>:0): Cannot index number with "foo"
$ echo $status
5
```

It aborts the whole program, so one bad line poisons every other line in the
same invocation. Guard with `select(type == "object")` immediately after the
`fromjson?`.

**Python.** `json.loads("42")` returns `42`, and `entry.get("type")` on it
raises `AttributeError` — the `except json.JSONDecodeError: continue` around
the parse does not catch it. Guard with `isinstance(entry, dict)`.

The blast radius is what makes this worth guarding against a line nobody has
seen. In a hook under `set -euo pipefail` the error takes the hook down on
*every* invocation for as long as the line sits in the read window — a
`tail -c` window means until it scrolls out — and a hook that dies is a feature
that silently stops working, not an error anyone is shown.

There is no published schema to appeal to ([[session-jsonl-schema]]), so "the
harness only ever writes objects" is an assumption about a format that is
conventional rather than specified.
