---
name: markdown-formatter-choice
description: "picking a prose hard-wrapper: judge by HTML render diff, not text diff; rumdl MD013 reflow has zero render changes, prettier can't keep `*em*`, dprint dedents fenced code, mdslw/remark split code spans and wrap before a literal `+`; `reflow = true` alone only breaks over-limit lines and leaves narrower wrapping as it is (a ~70-col doc comes out *longer*) — `reflow-mode = \"normalize\"` is what re-fills paragraphs, and a line-count cap is only honest against it; wrapping breaks line-grepping doc checkers; gating on it: `rumdl fmt --check`, never `rumdl check` (permanently red on an unwrappable line), and `include = [\"*.md\"]` crosses `/` — root-level files are `/*.md`"
metadata:
  node_type: memory
  type: reference
  originSessionId: 653ffbf7-f68c-4579-bda5-017506d33c78
  modified: 2026-09-02T15:22:51.886Z
---

Judge a markdown re-formatter by rendering every file to HTML with cmark-gfm
before and after and diffing the result, whitespace-collapsed. A token-level
or whitespace-insensitive text diff passes changes that alter rendering: a
line broken before a `>&2` or a `+ …` — inside a code span or in plain
prose — so it opens a blockquote or list item, a sentence split inside a
heading or table cell, a code-span edge space dropped. The render diff
also finds pre-existing malformed markdown — a three-backtick fence
nested in a three-backtick fence — which every re-serializer
re-interprets differently, and which is a source fix, not a tool
criterion.

Measured on ~100 files of design docs and plans, hard-wrapping at 80:

- **rumdl** (`enable = ["MD013"]`, `reflow = true`; `headings`,
  `code-blocks`, `tables = false` to silence what it cannot wrap): zero
  rendering changes on well-formed input, emphasis markers untouched,
  idempotent. **`reflow-mode` decides whether a pass normalizes the tree, and
  the default does not**: it only breaks lines already over the column and
  leaves a paragraph hand-wrapped narrower exactly as it is, so widths never
  converge and a doc hand-wrapped at ~70 came out 2 lines *longer*.
  `reflow-mode = "normalize"` re-fills every paragraph, which is what a
  line-count cap has to be measured against — prose wrapped at 60 sits at
  ~1.3x the line count of the same words at 80 and passes a cap it should
  not. Measured switching one tree: 85 lines changed under the default, 276
  across 31 files under `normalize`. `rumdl fmt` exits 0 with unwrappable
  lines left; `check --fix` exits 1. Its per-file cache buys nothing at that size (0.37 s vs 0.40 s):
  `--no-cache`, or a `.rumdl_cache/` dir appears in the tree.
- **prettier** `--prose-wrap always`: rewrites `*em*` → `_em_` with no option
  to keep it; one bold whose first word was a code span starting `**` broke.
- **dprint** markdown (`emphasisKind`/`strongKind: asterisks`): keeps
  emphasis, but dedents fenced code block content to the fence's indent
  and strips a one-sided space inside code spans.
- **mdformat** (+gfm): `---` rules become underscores (hardcoded), ordered
  lists renumber to `1. 1. 1.`.
- **mdslw**, **remark** + `@webpro/remark-wrap-prose`: break inside code
  spans / headings / table cells, and wrap a prose line so it starts
  `+ …`, which re-parses as a list item and truncates the paragraph —
  prettier, dprint and rumdl all avoid leaving a list marker at line
  start. remark is not idempotent.
- **pandoc** `-t gfm --columns`: reflows and drops code fences, escapes
  `#` and `**`.

**Wiring rumdl into a gate** (measured on rumdl 0.2.74):

- The non-fixing gate is `rumdl fmt --check -q --no-cache`: silent exit 0 on a
  clean tree, exit 1 with a unified diff when `fmt` would change a file.
  `rumdl check` is not usable as the gate — a line it cannot wrap (one long
  code span, a URL) is reported on every run, so it is permanently red.
- `include = ["*.md"]` is **not root-anchored**: the `*` crosses `/` and pulls
  in every tree (`plans/`, `memory/`, `.claude/`). Root-level files are
  `"/*.md"`.
- A line guard measured against the reflow needs the formatter check *ahead* of
  it in the same gate, and any ceiling taken from post-reflow counts — a file
  carrying 300-character lines gains most of its line count from the reflow,
  not from new words.

Every wrapper adds a `>` prefix to each newly wrapped continuation line of
a blockquote paragraph and realigns table separators; that is wrapping, not
a change. A wrapper also breaks any line-oriented doc checker that greps a
bullet for content — join a `- ` bullet with its indented continuation lines
before matching (see [[uv-direnv-venv]] for how rumdl gets installed).
