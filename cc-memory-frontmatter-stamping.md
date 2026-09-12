---
name: cc-memory-frontmatter-stamping
description: "who writes `node_type`/`originSessionId`/`modified`: one function, `stampNewMemoryContent`, on the Edit/Write/memory-write handlers only — never a Bash write; `originSessionId` is written only when absent, so it names the first tool write, not the creator; skips log `not stamping provenance`/`not dating` at warn"
metadata: 
  node_type: memory
  type: reference
  originSessionId: bcf16597-27f2-484f-9bfd-f206ec36dce5
  modified: 2026-09-05T19:39:59.272Z
---

A memory file's `metadata:` block — `node_type: memory`, `originSessionId`,
`modified` — is written by one function in the Claude Code bundle, which its own
log messages call `stampNewMemoryContent`. It is a pure content transform run
synchronously inside three tool handlers before the bytes reach disk: `Edit`,
`Write`, and the memory-store `write` tool (that one only for a `personal`
store). No file watcher, no debounce, no post-write pass, and no other emitter —
the block serializer has exactly one call site.

- **A `Bash` write never stamps.** Not a subprocess (`sed -i`, a `python3`
  heredoc), and not the in-process `_simulatedSedEdit` path either.
- **`originSessionId` is written only when the key is absent**, so it is never
  overwritten. It names the first session to write the file *through one of the
  three tools* — the creator only if the file was created that way. A file
  written by `Bash` and later edited by the tool carries the editor's id.
- **`modified:` is not restamped on every `Edit`/`Write`.** The stamper returns
  the content untouched, at `warn` or silently, when: the file is not `.md`; its
  path is not a *lexical* prefix match under the memory root (no realpath, so a
  symlinked memory dir is skipped); the new content does not open with a `---`
  fence at byte 0; the frontmatter carries a `rewriteHazard`; the `metadata:`
  mapping is inline (`{…}`) rather than block-style; or the rewrite fails its own
  faithfulness re-parse. A file carrying none of the fields can come through an
  `Edit` under the memory root and acquire none of them, with only a `warn` line
  to show for it.
- Files under `<memory>/team/` get the `modified:` rewrite but never
  `originSessionId` or `node_type`. `MEMORY.md` is inside the stamping scope and
  is spared only for lacking a frontmatter fence — give it one and it starts
  being stamped.

**Why:** the fields read as provenance some tool maintains, and the misreading
has been filed as a defect against gitlore's index→frontmatter sync, which
rewrites the first `description:` line and passes every other line through.

**How to apply:** read git history, not `modified:`, to learn who last changed a
memory file — a `Bash` edit leaves it untouched, so the field is a lower bound.
Read `originSessionId` as neither the last editor nor the creator. Before filing
a frontmatter write against a tool, `git log -S <field> -- <file>` in the store
settles whether that tool ever wrote it.

Read from the `2.1.261` bundle, the stamper also present in 2.1.258–2.1.260, and
consistent with 13 `ddaanet` facts edited in one session on 2026-09-02.
