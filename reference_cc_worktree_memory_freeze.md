---
name: reference-cc-worktree-memory-freeze
description: "`EnterWorktree` chdirs, never moves projectRoot: `CLAUDE_PROJECT_DIR`/`PATH`/`autoMemoryDirectory` stay at launch repo, stranding auto-memory; cwd≠`CLAUDE_PROJECT_DIR` = D15 drift signal"
metadata:
  node_type: memory
  type: reference
  originSessionId: 6fdf0ecf-96bd-4ca8-aab2-d54dc98d915a
  modified: 2026-07-27T21:41:44.034Z
---

Claude Code's in-process worktree feature (the `EnterWorktree` tool) is incompatible
with gitlore's launch-time `--settings autoMemoryDirectory` redirect (D10). Established
empirically 2026-06-10 (CC 2.1.170); re-verified against the 2.1.219 bundle 2026-07-24:

- **`autoMemoryDirectory` must be absolute** — the resolver expands a leading `~/`
  and otherwise rejects any non-absolute path outright, falling back to the default
  memory location. It is read from settings (policy, then the launch `--settings`
  file, then local/project when trusted, then user), so the `--settings`-supplied
  value is fixed for the process. No hook output field sets it.
- **`EnterWorktree` stays in the same process.** It `chdir`s to the worktree and
  updates the session's `originalCwd`, but does **not** touch the session
  `projectRoot` — which is what CC exports to hooks as `CLAUDE_PROJECT_DIR`. PATH
  (front = launch repo's `.gitlore/bin`), `DIRENV_DIR`, and the launch `--settings`
  `autoMemoryDirectory` are process env / launch state that a `chdir` cannot move.
  So auto-memory written from the worktree lands on the LAUNCH repo's memory
  tree/branch, not the worktree's. (`ExitWorktree` does reset `projectRoot` when it
  restores the cwd.)
- **A fresh `claude` whose cwd is the worktree resolves correctly** — the shim
  recomputes `git rev-parse --show-toplevel`. The only broken path is the in-process
  tool; any fresh shell launch from the worktree dir is fine.
- **Detection (D15, SHIPPED):** hook payloads carry the live `cwd` and CC spawns the
  hook process in the live cwd (warning and falling back to the launch cwd only if it
  has vanished), while `CLAUDE_PROJECT_DIR` is the frozen `projectRoot`;
  `toplevel(cwd) != CLAUDE_PROJECT_DIR` (same git common-dir) flags the drift.
  Shim env (e.g. `GITLORE_LAUNCHED`) IS inherited by hooks, so an exported
  `GITLORE_LAUNCH_ROOT` would also serve as the exact memory-target marker.
- **`EnterWorktree` AND `ExitWorktree` both fire `PostToolUse`**, and a name-based
  `matcher:"EnterWorktree|ExitWorktree"` matches `tool_name` — so the targeted matcher
  wins over `"*"` (fires once per transition, no per-Bash cost, no de-dup). Payloads:
  `EnterWorktree` → `cwd`=worktree, `tool_response` `worktreePath`+`worktreeBranch`
  (+`message`); `ExitWorktree` → `cwd`=restored launch root, `tool_response`
  `originalCwd`+`worktreePath`+`worktreeBranch`+`message` (plus `tmuxSessionName` or
  `discardedFiles`/`discardedCommits` depending on branch). `action` (`keep`/`remove`)
  is an ExitWorktree **tool_input** parameter, not a response field. The
  Enter-warns/Exit-silent asymmetry falls out of the predicate for free.
  See [[reference-stale-plugin-code]].
- **Conversation namespace is per sanitized cwd**, so CC worktrees get their own
  `~/.claude/projects/<encoded>/` (worktrees live under `.claude/worktrees/<name>`).
  `--continue`/`-c` is cwd-project-scoped (won't find a parent-dir session from a
  worktree); `--resume <id>` resumes by id; `--fork-session` forks. CC has native
  `--worktree`/`--tmux` flags.
- **Worktree creation can be hook-provided.** `WorktreeCreate`/`WorktreeRemove` hooks
  supply the path for VCS-agnostic isolation, `EnterWorktree` takes a `path` to enter
  an existing worktree, and subagents accept `isolation:"worktree"` — all of which
  reach the same in-process `chdir`, so the env-freeze above applies to every route.
  See [[reference-hook-input-schema]].

See [[reference_cc_project_dir_encoding]], [[reference-hook-output-channels]];
the launch-time `--settings autoMemoryDirectory` redirect itself is D10 in
`docs/design.md`.
