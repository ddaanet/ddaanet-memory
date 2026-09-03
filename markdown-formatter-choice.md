---
name: markdown-formatter-choice
description: "picking a prose hard-wrapper: judge by HTML render diff, not text diff; rumdl MD013 reflow has zero render changes, prettier can't keep `*em*`, dprint dedents fenced code, mdslw/remark split code spans; wrapping breaks line-grepping doc checkers"
metadata:
  type: reference
---

Judge a markdown re-formatter by rendering every file to HTML with cmark-gfm
before and after and diffing the result, whitespace-collapsed. A token-level
or whitespace-insensitive text diff passes changes that alter rendering: a
line broken inside a code span so `>&2` or `+ …` opens a blockquote or list
item, a sentence split inside a heading or table cell, a code-span edge
space dropped. The render diff also finds pre-existing malformed markdown —
a three-backtick fence nested in a three-backtick fence — which every
re-serializer re-interprets differently, and which is a source fix, not a
tool criterion.

Measured on ~100 files of design docs and plans, hard-wrapping at 80:

- **rumdl** (`enable = ["MD013"]`, `reflow = true`; `headings`,
  `code-blocks`, `tables = false` to silence what it cannot wrap): zero
  rendering changes on well-formed input, emphasis markers untouched,
  idempotent. `rumdl fmt` exits 0 with unwrappable lines left; `check --fix`
  exits 1. Its per-file cache buys nothing at that size (0.37 s vs 0.40 s):
  `--no-cache`, or a `.rumdl_cache/` dir appears in the tree.
- **prettier** `--prose-wrap always`: rewrites `*em*` → `_em_` with no option
  to keep it; one bold whose first word was a code span starting `**` broke.
- **dprint** markdown (`emphasisKind`/`strongKind: asterisks`): keeps
  emphasis, but dedents fenced code block content to the fence's indent
  and strips a one-sided space inside code spans.
- **mdformat** (+gfm): `---` rules become underscores (hardcoded), ordered
  lists renumber to `1. 1. 1.`.
- **mdslw**, **remark** + `@webpro/remark-wrap-prose`: break inside code
  spans / headings / table cells; remark is not idempotent.
- **pandoc** `-t gfm --columns`: reflows and drops code fences, escapes
  `#` and `**`.

Every wrapper adds a `>` prefix to each newly wrapped continuation line of
a blockquote paragraph and realigns table separators; that is wrapping, not
a change. A wrapper also breaks any line-oriented doc checker that greps a
bullet for content — join a `- ` bullet with its indented continuation lines
before matching (see [[uv-direnv-venv]] for how rumdl gets installed).
