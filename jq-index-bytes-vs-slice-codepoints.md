---
name: jq-index-bytes-vs-slice-codepoints
description: "a jq slice cutting mid-character on non-ASCII, or a computed offset matching nothing: `index()` counts BYTES, slicing and `length` codepoints; use `split($sub)[0]|length`; `--rawfile` for whole files"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 826ecad8-b84e-451e-ae20-0a6b9308f8ec
  modified: 2026-08-14T16:23:46.601Z
---

jq's `index($sub)` on a **string** haystack returns a **byte** offset, while
`$s[$a:$b]` slicing and `$s | length` count **codepoints**. Feeding an `index`
result into a slice holds only while the text before the match is pure ASCII,
and cuts inside a character as soon as it is not. Nothing warns: the result is
a mangled string, or an offset that matches nothing.

One command shows the two units disagreeing:

```
$ jq -cn '"— A\n— X\n" as $s | "\n— X" as $old
  | {index: ($s|index($old)), slice_needs: ($s|split($old)|.[0]|length)}'
{"index":5,"slice_needs":3}
```

Take the offset from the same unit system as the slice — the prefix `split`
leaves behind:

```
($s | split($old)) as $parts
| if ($parts|length) < 2 then empty          # no occurrence
  else ($parts[0] | length) as $i | ...      # codepoints, like the slice
  end
```

`split/1` is literal; only `split/2` is a regex. `$parts[0]` is the text before
the *first* occurrence, which is the semantics a replace-style computation
wants. Verified against jq 1.7.

The reason to be in jq for this at all is `--rawfile`, which binds a whole file
as one string with its final byte intact — `$(cat f)` strips the trailing
newline, and in a file format where a newline is the separator that byte is
usually the one that matters. See [[parse-dont-regex-structured-formats]].
