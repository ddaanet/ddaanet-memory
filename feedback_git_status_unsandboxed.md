---
name: git-status-unsandboxed
description: "sandbox materializes home dotfiles into cwd; run `git status`/`diff` with `dangerouslyDisableSandbox: true` so the output isn't 14 lines of noise"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6924c56d-d077-4b09-abbe-4018a9e87dc0
---

Run `git status` unsandboxed (`dangerouslyDisableSandbox: true`). Same for any
diff/state command whose output you intend to act on.

**Why:** The sandbox materializes a bunch of home-directory dotfiles
(`.bash_profile`, `.bashrc`, `.gitconfig`, `.profile`, `.ripgreprc`,
`.zprofile`, `.zshrc`, etc.) and other view-only entries into the working
directory when git commands run inside it. They appear as untracked in `git
status` and pollute the signal. The user has to scroll past noise to find the
two real lines they care about. Observed concretely on 2026-05-19 in
`/Users/david/code/handoff` — sandboxed run reported 16 untracked entries;
unsandboxed run reported 2.

**How to apply:** Any time you call `git status`, `git diff`, `git ls-files
--others`, or any command whose output the user will read to decide what's
actually in their working tree, pass `dangerouslyDisableSandbox: true`. Don't
add explanatory text apologising for the noise — just skip the noise.

**Slash-command context is sandboxed too.** Commands like
`/commit-commands:commit` auto-inject a `git status` into their `## Context`
block — that injection runs sandboxed, so it lists the home dotfiles as
untracked and can read as "nothing staged, untracked files present" when the
tree is actually clean. Never act on that injected listing directly: re-run
`git status` unsandboxed first. Never stage/commit the materialized dotfiles
(`.bashrc`, `.gitconfig`, `.zshrc`, …) — committing them pollutes the repo
with personal config. Confirmed 2026-05-24: a `/commit` context showed ~14
untracked dotfiles + `scripts/__pycache__/`; the unsandboxed tree had only the
build artifact, nothing to commit.
