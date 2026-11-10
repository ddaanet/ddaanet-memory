---
name: rumdl-unwrappable-span
description: "`[MD013] Line length N exceeds 80` left by `rumdl fmt` on a long code span or `*`/`**` emphasis run: fmt re-joins a hand split; move the span into a fenced block — punctuation after the closing backtick is lost and a hand-wrapped span's newline becomes a real break"
metadata:
  node_type: memory
  type: reference
  originSessionId: 850fe098-080d-45c3-8e27-54f5fe2fc808
  modified: 2026-11-10T13:34:25.689Z
---

Measured on rumdl 0.2.60 with `reflow-mode = "normalize"`.

`rumdl fmt` treats a code span and an emphasis run (`*…*`, `**…**`) as one
unbreakable unit. One wider than the column is reported as
`[MD013] Line length N exceeds 80` on every run while `fmt` exits 0, and a split
made by hand across source lines is re-joined onto one line by the next `fmt`.
Splitting it never clears the warning.

A long code span clears by moving it into a fenced code block, which MD013
exempts under `code-blocks = false`. A text diff that ignores whitespace and
punctuation passes the move while missing both of its defects:

- Punctuation right after the span's closing backtick has nowhere to go, since
  a fence's closing line holds nothing else. A comma or semicolon that joins
  clauses moves to the start of the following prose, or the sentence is
  rephrased; dropping it changes the sentence.
- A span hand-wrapped across source lines renders each newline as a space, and
  a fence keeps the newline. Join the fence content onto one line, or a quoted
  regex or message gains a line break it never had.

An emphasis run has no exempt form: it stays over the column unless its markup
changes.

The gate and its configuration are in [[markdown-reflow-gate]].
