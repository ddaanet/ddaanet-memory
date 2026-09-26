---
name: markdown-reflow-gate
description: "wiring a markdown hard-wrap gate (rumdl 0.2.74): the gate is `rumdl fmt --check -q --no-cache`, never `rumdl check` — permanently red on a line it cannot wrap; `include = [\"*.md\"]` is not root-anchored, the `*` crosses `/` and pulls in `plans/`, `memory/`, `.claude/` — root-level files are `/*.md`; without `--no-cache` a `.rumdl_cache/` dir appears in the tree; `reflow = true` alone only breaks over-limit lines and never converges widths (a ~70-col doc comes out *longer*) — `reflow-mode = \"normalize\"` re-fills paragraphs and is what a line-count cap is measured against, formatter check ahead of the line guard; reflow adds `>` to wrapped blockquote continuations, realigns table separators, and breaks any checker that greps a `- ` bullet line for content"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 71b27bc8-eb7a-4d9a-ae83-506402f30720
  modified: 2026-09-26T12:27:54.540Z
---

Measured on rumdl 0.2.74. The working config is `enable = ["MD013"]` with
`reflow = true`, and `headings`, `code-blocks` and `tables = false` to silence
what MD013 cannot wrap. [[uv-direnv-venv]] covers getting rumdl onto `PATH`.

## `reflow-mode` decides whether a pass normalizes, and the default does not

`reflow = true` alone breaks only lines already over the column, leaving a
paragraph hand-wrapped narrower exactly as it is. Widths never converge: a
document hand-wrapped at ~70 columns comes out 2 lines *longer*.
`reflow-mode = "normalize"` re-fills every paragraph — measured switching one
tree, 85 lines changed under the default, 276 across 31 files under `normalize`.

A line-count cap is honest only against `normalize`, since prose wrapped at 60
sits at ~1.3x the line count of the same words at 80. So the formatter check
runs *ahead* of the line guard in the same gate, and any ceiling is taken from
post-reflow counts — a file carrying 300-character lines gains most of its line
count from the reflow, not from new words.

## The gate is `fmt --check`, never `check`

`rumdl fmt --check -q --no-cache` is the non-fixing gate: silent exit 0 on a
clean tree, exit 1 with a unified diff when `fmt` would change a file.

`rumdl check` is unusable as a gate — a line it cannot wrap (one long code span,
a URL) is reported every run, so it is permanently red. The two subcommands also
disagree there: `rumdl fmt` exits 0 with unwrappable lines left, `check --fix`
exits 1.

`--no-cache` is not an optimisation. The per-file cache buys nothing at ~100
files (0.37 s vs 0.40 s), and without the flag a `.rumdl_cache/` directory
appears in the tree.

## `include` is not root-anchored

In `.rumdl.toml`, `include = ["*.md"]` does **not** mean root-level files: the
`*` crosses `/` and pulls in every tree, including `plans/`, `memory/` and
`.claude/`. Root-level files are `"/*.md"`.

## What a reflow does downstream

Two diff effects are wrapping rather than edits, and a review flagging them is
misreading the diff: a `>` prefix added to each newly wrapped continuation line
of a blockquote paragraph, and realigned table separators.

A reflow breaks any checker that greps a single line for content, because a
`- ` bullet's content moves onto indented continuation lines — join a bullet
with its continuations before matching. A field-per-line layout is re-filled
into prose: [[reflow-joins-field-lines]].
