---
name: gitlore-memory-in-git
description: memory/ directory is tracked by gitlore and belongs in git commits — do not gitignore it
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9440f7e6-f4d9-466b-b3b6-0d1e988b7a7a
---

The `memory/` directory is managed by gitlore, which intentionally puts session notes (auto-memory) in git. It is part of the repo and should be committed alongside code changes.

**Why:** gitlore's whole point is versioned session notes in git.

**How to apply:** Never add `memory/` to `.gitignore`. When running preflight or preparing a commit, include `memory/` in the staged files like any other tracked directory.
