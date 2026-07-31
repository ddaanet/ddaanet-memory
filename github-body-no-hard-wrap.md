---
name: github-body-no-hard-wrap
description: "writing PR/issue/comment bodies for GitHub — don't hard-wrap prose, skip a files-changed list"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c4685bd7-4ba3-4a0c-aba9-3196e2134b24
  modified: 2026-07-30T15:35:21.896Z
---

When authoring GitHub PR / issue / comment bodies, do NOT hard-wrap prose paragraphs
at a column width, and do NOT include a hand-written "files changed" list.

**Why:** GitHub Flavored Markdown in comment fields renders every single newline as a
visible line break (`<br>`), so column-wrapped paragraphs show as jaggedly broken lines
instead of flowing text. And a manual files list is redundant with the PR's "Files
changed" tab (which is canonical and never goes stale). my human partner trimmed both from
micro PR #4142.

**How to apply:** Write one physical line per paragraph (let the field soft-wrap);
keep blank lines only between paragraphs / list items / headings. Rely on the Files
changed tab instead of enumerating files in the body. Blank lines around lists and
headings are still fine — the rule is about mid-paragraph newlines.
