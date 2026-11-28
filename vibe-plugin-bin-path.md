---
name: vibe-plugin-bin-path
description: "Vibe plugin `bin/` is never on PATH; the skill load's base directory is the anchor; `.git` is ignored in the plugin digest"
metadata:
  node_type: memory
  type: reference
  originSessionId: f16a6bed-a953-4e46-94d8-6ea12bb015d3
  modified: 2026-11-28T14:26:50.033Z
---

Pinned 2026-11-25 against the installed mistral-vibe source
(`vibe/core/plugins/_compatibility.py`, `_native.py`, the
`mistralai_vibe_local_harness` hooks runner) and a
`claude plugin validate --strict` probe.

## Skills: no plugin bin on PATH

A plugin's `bin/` directory exposes nothing in Vibe (no PATH injection, no
tool surface). The anchor is the base directory text Vibe prints when a skill
loads — skill bodies that must run plugin scripts invoke
`<base>/../../bin/<name>`. `.git` is in the universal runtime-state ignore
set, so a plugin directory that is a `--branch dist-vX.Y.Z` clone digests as
its files alone: the clone itself is the snapshot install.
