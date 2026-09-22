---
name: reflow-joins-field-lines
description: "a bullet with one field per continuation line (`Requirements:`, `Model:`, `Change:`) is one paragraph to markdown, so `just format-docs`/rumdl MD013 re-fills it into prose; write a field list as a nested list"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 78c01ada-e3fc-450c-b044-3f914c0034dc
  modified: 2026-09-22T13:05:24.723Z
---

Markdown has no field-list construct. A bullet whose indented continuation
lines each hold a field is, to the spec and to every reflowing formatter, one
paragraph — and a reflow (rumdl MD013 `reflow-mode = "normalize"`, run by
`just format-docs`) re-fills it into running prose, leaving the field names
mid-line where a line-reading parser finds none. The formatter is operating
correctly; the document was relying on line breaks the format does not
preserve.

The markdown construct that does keep one field per line is a list: reflow
never merges list items, so a nested list survives.

```markdown
- Item 2.1 — the guard's initial-release branch
  - Requirements: FR3
  - Model: sonnet
  - Change: …
```

**How to apply:** write a field list as a nested list in any document a
formatter runs over, and run the formatter over a format template once before
adopting it — a template written before the formatter existed is the usual
carrier of the continuation-line shape. Formatter selection and its other
artefacts are in [[markdown-formatter-choice]].
