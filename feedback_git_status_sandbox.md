---
name: feedback_git_status_sandbox
description: "Run `git status` (only that command) with the CC command sandbox disabled — sandboxed, it reports phantom untracked dotfiles"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ca0e5a2e-2efc-4d50-bc14-b55050e722d0
  modified: 2026-07-24T12:24:41.204Z
---

Run `git status` with the command sandbox disabled
(`dangerouslyDisableSandbox: true`) in any repo. Only `git status` — other git
commands follow the usual sandbox rules.

**Why:** under CC's command sandbox, `git status` reports phantom untracked
entries — the sandbox bind-mounts `/dev/null` (char device `1,3`) over the
host's dotfiles (`.bashrc`, `.zshrc`, `.gitconfig`, `.idea`, …), and git sees
those phantom paths as untracked. Run unsandboxed they vanish, giving an
accurate status. (Separately, `.git` writes can fail "Device or resource busy"
sandboxed — but the status-pollution is the reason to run git unsandboxed.)

**How to apply:** pass `dangerouslyDisableSandbox: true` on Bash calls that run
`git status`. Leave other git commands sandboxed as usual. The same phantom
untracked entries also show up in the `/commit-commands:commit` skill's context
block (it gathers `git status` sandboxed, and you can't disable the sandbox for
a skill's internal calls) — so when committing, stage real changes by explicit
path; never trust that untracked list with a blanket `git add -A`. Related:
[[reference_sandbox_git_lockfile]], [[feedback_strict_sandbox_git]].
