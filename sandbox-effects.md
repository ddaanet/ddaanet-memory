---
name: sandbox-effects
description: "index.lock/\"Another git process\" (mask vs genuinely-stale: only a real lock survives unsandboxed), phantom dotfiles as char devices or zero-byte files (break `git add -A`; real while ANY agent's sandbox is live, so one clean reading proves nothing), sibling worktree \"Read-only file system\", \"Device or resource busy\" on .claude/settings.json & .git/config (blocks git checkout/reset --hard), own hooks.json unlink EROFS, --resume \"No conversation found…\", $TMPDIR unset, zellij \"no active session\", empty `ps` for background tasks, sandboxed `claude -p` silently drops every SessionStart hook, `apply-seccomp: unshare(CLONE_NEWUSER)` = sandbox init failed outright, /tmp \"No space left\"/\"Out of diskspace\" with du≪df = other sandboxes pin the tmpfs (clear /tmp/claude-1000 caches); escapes incl. dangerouslyDisableSandbox and `excludedCommands` (`git *` = legacy `git:*`; EVERY segment must match since 2.1.277, so `| head`, `cd`, `> file`, `git -C` stay sandboxed; does NOT auto-approve)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-08-26T07:38:31.000Z
---

Independent mechanical facts that share one trigger surface: the Claude Code
command sandbox distorting git, filesystem, transcript and IPC operations. Each
section stands alone and is keyed by the symptom an agent arrives holding.

**Baseline the rest of the file assumes:** `/sandbox` set to *unsandboxed
fallback* (not strict), auto mode on, and user-scope
`sandbox.excludedCommands: ["git *", "find *", "ls *", "claude *", "just release *"]`
(legacy `git:*` spelling equivalent). That runs a call outside the sandbox only
when *every* segment in it matches an entry (CC ≥ 2.1.277), so the sections
below are met wherever the exclusion does not reach: a pipeline to `head`,
`cat` or `wc`, a `cd` anywhere in the call, a redirect to a file, `git -C`,
a command nested in `( … )`, `$( … )` or `sh -c`, a `just` recipe or script
that runs git internally, or a machine without the entries.

| symptom | section |
|---|---|
| `Unable to create '.git/index.lock': File exists` | `.git/index.lock` |
| `.bashrc`/`.zshrc`/`.claude/skills` as untracked; `can only add regular files` | Phantom dotfiles |
| a slash command's `## Context` lists phantom dotfiles | Phantom dotfiles |
| `Device or resource busy` / `Read-only file system` on `.claude/settings.json`, `.git/config`, `hooks/hooks.json` | Harness-mounted config paths |
| `Read-only file system` from `git -C <other-worktree>`, `cp`/`mv` into another repo | Sibling worktree writes |
| `$TMPDIR` expands empty, `/foo.log: Permission denied`; a benign compound denied | Leaving the sandbox |
| `ps` shows nothing but your own shell | `ps` |
| `No conversation found with session ID` | `--resume` |
| `There is no active session!` from zellij | zellij |
| a `SessionStart` side effect never happened after `claude -p` | `claude -p` |
| `apply-seccomp: unshare(CLONE_NEWUSER): Invalid argument` | Sandbox setup fails |
| `No space left on device` / `Out of diskspace` under `/tmp`, `du` far below `df` | `/tmp` reads full |

## `.git/index.lock`: one error string, three causes

`fatal: Unable to create '.git/index.lock': File exists. Another git process
seems to be running` names a concurrent process that usually does not exist.
Three causes share the string, and the correct action is opposite between them,
so run the discriminator before touching anything:

1. **Run the existence test unsandboxed** — `ls -la .git/index.lock` with
   `dangerouslyDisableSandbox: true`.
2. **Path absent unsandboxed → a mask.** The sandbox can present
   `.git/index.lock` itself as a masked path, so `git commit`/`git add` fail
   over a lock that does not exist. Tells: the message is self-contradictory
   (creation *failed*, yet a file is *there*), a zero-byte lock, no git process
   in `ps`, a sandboxed `ls` that showed nothing a moment earlier. Leave it
   alone and re-run the command unsandboxed. This can start mid-session;
   earlier commits in the same session went through sandboxed.
3. **Path absent, but the failure came right after a sandboxed `git add` → a
   stranded lock.** A `git add` inside the sandbox can stage successfully while
   failing to remove the lock; the *next* command's `pre-commit` hook runs its
   own `git add`, cannot take the lock, and the commit aborts — after which git
   cleans the lock, so the evidence is destroyed by the time you look. Observed
   2026-07-21 in gitlore on `git add <file> && git commit` in one sandboxed
   call. Re-running the identical command unsandboxed succeeds.
4. **Path persists unsandboxed, no process holds it (`fuser` empty), an
   unsandboxed write fails the same way → genuinely stale.** `rm` it; the write
   recovers and it does not reappear. On 2026-07-23 a `.git/config.lock` looked
   exactly like a mask (`-r--r--r-- … 0`, synthetic-identical
   birth/mtime/ctime) and was real: an orphan from a `git config` write killed
   mid-rename two days earlier, plausibly OOM, surfacing as `could not lock
   config file .git/config: File exists` from gitlore's `SessionStart` hook.

The 0-byte/read-only shape alone proves nothing. Deleting a lock you have not
proven stale is the dangerous move, and the sandbox makes an entirely healthy
repo look like a crashed one.

**Do not "fix" the hook in response.** `GIT_TRACE=1 git commit` unsandboxed
prints `run_command: GIT_EDITOR=: GIT_INDEX_FILE=.git/index …/hooks/pre-commit`,
proving git has NOT taken `index.lock` when `pre-commit` runs, so a hook's bare
`git add` is legitimate. A staging hook's `GIT_INDEX_FILE` capture/restore
([[git-hook-env-leak]]) is a different bug with the same error string: that one
fires under `-a`/pathspec commits where git really does hold the lock, and it
fails sandboxed or not — an unsandboxed retry succeeding is what rules it out.

## Phantom dotfiles: the `/dev/null` masks

The sandbox bind-mounts `/dev/null` over a closed set of paths in the
**session's project root** — every file a repository could use to hijack
execution. Measured 2026-08-15 in gitlore by diffing sandboxed listings against
unsandboxed ones, 22 paths stat as `crw-rw-rw- nobody nogroup 1, 3`:

```
.bash_profile .bashrc .gitconfig .idea .mcp.json .profile .ripgreprc
.vscode .zprofile .zshrc
.claude/{agents,commands,hooks,launch.json,loop.md,output-styles,routines,
         scheduled_tasks.json,skills,workflows}
.git/config.lock .git/config.worktree
```

Scope is the project root, not the cwd: a listing of a subdirectory is clean
with the cwd set there, and `find -maxdepth 1 -type c` returns zero in other
repo roots, so cross-repo commands are unaffected and a deny predicate can be
anchored to `CLAUDE_PROJECT_DIR`. The two `.git/` entries are why `git
checkout`/`reset --hard` hit `Device or resource busy` on `.git/config` (next
section). A second masking shape exists: since 2026-07-20 deny-listed paths have
also appeared as **zero-byte read-only regular files** (`-r--r--r-- … 0`), a
path that ought to be a directory getting a zero-byte file too.

**Every masked name begins with a dot, which decides who sees them.** Plain
`ls` shows none; `ls -a` shows them; `find` lists them unconditionally; `rg`
shows none (hidden skipped by default); `grep -r` finds nothing in them, since
a `/dev/null` mask reads as immediate EOF; `cat`/`head` of one returns empty.
So the corrupted commands are `ls -a`, `find`, `du`, `tree` and the `git status`
family — `git status` reports them as `??`, and sandboxed `git ls-files
--others --exclude-standard` reported 19 untracked paths in gitlore where the
truth was 3. Nothing in the output marks them as artefacts.

**They break `git add -A` under the sandbox**: `error: .bash_profile: can only
add regular files, symbolic links or git-directories`, and the whole `add`
aborts. Stage explicit pathspecs. (`git add -A` is prohibited on its own merits
regardless; and an explicit pathspec still strands the lock sandboxed, per the
section above — the exclusion is what fixes both.)

**They can be visible with the sandbox off.** Unsandboxed readers — via
`dangerouslyDisableSandbox` and via `excludedCommands` — have seen the masks
while some other agent was mid-`Bash` call: a controller's own unsandboxed `git
status` showed a clean tree one moment and fourteen phantom dotfiles the next
with parallel subagents live. The observation is repeated; the causal reading
("real for every process while any sandbox is live") is **not established** — a
2026-08-16 probe saw an unsandboxed `!`-expansion return the full mask listing
once with a sandboxed background task live, and three later runs under the same
conditions were clean. Treat it as: one clean reading proves nothing about a
later moment, and presence says nothing about the repo. Do not delete them, do
not commit them (committing pollutes the repo with personal config), do not
raise them as a finding.

**A slash command's `## Context` block is a Bash call.** The injection is
`` !`cmd` `` expansion in the command body (`commit-commands/commands/commit.md`
carries `` - Current git status: !`git status` ``), and decompiling CC 2.1.233
shows it executed through the ordinary Bash tool `call` with a bare
`{command}` — so the sandbox decision and the `excludedCommands` match apply
exactly as to a typed call (confirmed live 2026-08-16: an excluded `git status`
expansion saw the real tree, an unmatched one saw the masks), while
**`PreToolUse` hooks do not run on that path**, and several `` !` `` blocks in
one body run concurrently. Under `git:*` the listing is real. Without it, never
act on the injected listing: re-run `git status` unsandboxed first (a
2026-05-24 `/commit` context showed ~14 untracked dotfiles where the
unsandboxed tree had one build artefact).

## Harness-mounted config paths are busy/read-only

The harness mounts certain paths as busy/read-only/char-devices:
`.claude/settings.json`, `.claude/settings.local.json`, `.git/config`,
`.mcp.json`, `.gitconfig`, `.gitmodules` and similar, and the sandbox denies
writes to `.claude/settings.json` outright. Any operation that unlinks or
rewrites one *in place* fails under the sandbox with `Device or resource busy`
or `Read-only file system`:

- `git checkout <branch>` and `git reset --hard` — cannot unlink or overwrite
  the protected files.
- `git branch -d` — warns "could not write `.git/config`"; the branch is
  deleted anyway.
- Tools doing an atomic `mktemp` + `mv` into these paths (e.g.
  `claude-plugin-dev`'s `install.sh` settings-merge), because the `mktemp`
  lands on a different device.
- **A plugin's own `hooks/hooks.json`, when it is the active hooks config for
  the session.** In gitlore's own repo a sandboxed `git checkout`/`git merge
  --ff-only` that changes `hooks/hooks.json` dies with `unable to unlink old
  'hooks/hooks.json': Read-only file system` — the live hooks config is held
  read-only regardless of the write allowlist; every other file in the same
  merge updates normally. Confirmed 2026-07-24; the identical command succeeded
  unsandboxed.

**How to apply:** run those operations unsandboxed (`git:*` does this for the
git forms). When `checkout` + `merge` choke on the protected files, a plain
`git reset --hard <branchtip>` unsandboxed completes the fast-forward cleanly.
Writes to `.claude/settings.json` go through the built-in **`update-config`**
skill ("configure the Claude Code harness via settings.json"): verified on CC
2.1.233 writing `sandbox.excludedCommands` to user scope via `Edit`, which the
classifier did not refuse. Whether that route also clears the `enabledPlugins`
denial ([[classifier-denied-self-config]]) is untested. The contrast stands
either way: a harness tool is not under the command sandbox, a raw git unlink
of the same path is.

**Recovering a merge interrupted mid-way by this failure:** a `git merge
--ff-only` that dies on `hooks/hooks.json` can leave the working tree
PARTIALLY updated to the target's content for files processed before the
failure, while HEAD stays at the pre-merge commit (git does not roll back files
it already wrote). Before retrying, `git status` and diff any "modified" files
against the merge target — if they already match it exactly, `git checkout
HEAD -- <files>`, remove any untracked new files the target added, THEN retry
the merge unsandboxed in one shot rather than resuming piecemeal.

## Sibling worktree writes

The sandbox's write allowlist covers the session's working directories plus
`.git` and nothing else. From a linked worktree, a git op writing a *sibling*
path fails with `Read-only file system` — `git -C <main-root> merge/push
<branch>` (finishing a worktree branch into main), `git worktree remove
<other>` — and so does a plain `cp`/`mv` into another repo. The merge
half-prints "Updating …" then aborts cleanly (no `MERGE_HEAD`). Retry
unsandboxed; where the user's own prompt asked for the write, the auto
classifier lets the retry through, so routing around the denial with a tool
that is not under the command sandbox buys nothing and loses the shell.

## Leaving the sandbox: the two escapes and what they cost

**Per call:** `dangerouslyDisableSandbox: true` on the Bash tool. **Standing:**
`sandbox.excludedCommands` in `~/.claude/settings.json` (mechanism in the next
section), honoured in strict mode too.

**Strict mode blocks the per-call flag.** With `/sandbox` set to *Strict
sandbox mode*, `dangerouslyDisableSandbox` is refused and git writes fail with
`Read-only file system` on `.git/index.lock`; the exclusions still apply. Ask
my human partner to run the command via `! git …` or to toggle the mode — do
not retry the same sandboxed command. Strict mode is a user setting and is
usually off in ddaanet repos: check rather than assume.

**The read-only-inspection default.** Reaching for `dangerouslyDisableSandbox`
to read local git state or a masked path is a standing default my human
partner set explicitly, not a per-case approval to seek: call it directly,
without pre-announcing or asking. It fires on a symptom rather than while
composing: a phantom dotfile in a listing, an empty `cat` of a `.claude/` path,
a `git status` in a call the exclusion did not cover (next section — a pipe,
`cd`, `git -C`, a redirect, `( … )`, `$( … )`, `sh -c` or a recipe), or a
segment the matcher missed (a double space, `/bin/ls`). Mutating operations (`git commit`,
`push`, `reset`, `checkout -- …`, anything that writes) are NOT covered and
follow normal confirmation rules.

**Cost 1 — `$TMPDIR` is unset when unsandboxed.** The sandbox is what exports
it, so `cmd > "$TMPDIR/foo.log"` expands to `/foo.log` and dies with
`Permission denied`. Use the absolute scratchpad path in any unsandboxed call.
Hit on `just release minor` (network for `git push`/`gh`, so the whole recipe
ran unsandboxed); the release went through once the log redirect used the
absolute path — including the cross-repo marketplace push, because the
classifier evaluates the top-level `just release minor`, not the inner
`git -C "$MARKETPLACE_DIR" push`.

**Cost 2 — an unsandboxed command reaches the classifier, and it denies benign
compounds unpredictably.** Sandboxed commands are auto-allowed
(`autoAllowBashIfSandboxed`) with no classifier call; leaving the sandbox by
either escape moves the command *into* classifier evaluation unless the
read-only validator (next section) clears it. Observed 2026-08-16 after the
exclusions were written: `ls -a | wc -l; echo …`, `find … | wc -l; echo …` and
`claude --version; echo …` were each refused while `ls -a; echo …`,
`find …; echo …`, `git log …; echo …` and a bare `claude --version` passed. No
stable rule separates them — the pipe is not it. Mitigation is shape, not
settings: keep an excluded command in its own call rather than appending a
compound to it. A classifier call is one model call over cached context with no
generation, cheaper than an agent turn, so batching `cd <E> && git status` into
one call still beats splitting it.

**Cost 3 — a recipe that must run unsandboxed must run *entirely* so.**
`.envrc` vars pass through the sandbox (`MARKETPLACE_DIR`, `DIRENV_*` — direnv
runs in the launching shell), but the variable arriving does not make the path
it names writable. A sandboxed `just release` gets through the bump, commit,
tag, `git push` and `gh release create`, then dies on the marketplace step:
`mv` of the `mktemp` file onto `$MARKETPLACE_DIR/.claude-plugin/marketplace.json`
reports `inter-device move failed … unable to remove target: Read-only file
system`. Every step before it is irreversible, so the plugin is left tagged and
published against a stale marketplace entry (confirmed 2026-07-30, gitlore
0.4.4). `just resume-release` (plugin-dev >= 0.5.0) recovers a half-landed one,
but only unsandboxed: its `bump_marketplace` runs that `mv` unconditionally,
before comparing local and remote heads. `git:*` does not reach a git call
inside a recipe — `just` is the top-level segment.

## `excludedCommands`: every segment must match

Decompiled from CC 2.1.233, re-checked against the 2.1.289 bundle, the
sandboxing and settings-reference docs, and probes (`${TMPDIR-UNSET}` is the
discriminator: `/tmp/claude-1000` sandboxed, `UNSET` not; phantom dotfiles in
an `ls -la` are another):

- **Entries are permission-rule patterns, not plain strings.** `foo bar` is
  exact; `foo bar *` (documented) and `foo bar:*` (legacy, still parsed as the
  same prefix) match `foo bar` with or without further arguments; any other
  unescaped `*` is a glob. A plain `"git status"` entry covers only
  the bare literal — 2.8% of real `git status` traffic across a 45-day
  transcript corpus, since `--short`, `--porcelain` and `git -C <dir> status`
  all miss it. Prefix entries anchor at the segment start, so `git status:*`
  cannot reach `git -C <dir> status` (31.6% of git-bearing calls); only `git *`
  covers the other shapes, which is why the baseline carries it. Since 2.1.277
  even `git *` no longer covers git's value-taking global options — `-C`, `-c`,
  `--git-dir`, `--work-tree` keep the call sandboxed (issue #95455, open);
  boolean ones like `--no-pager` still match.
- **Splitting** is a tree-sitter-bash parse, cut at `&&`, `||`, `|`, `;`, `&`
  and newline. Nesting is never descended into: `( … )`, `$( … )`, `sh -c '…'`
  and `if …; fi` are compared whole, so they never match — nesting fails closed,
  and so does a parse failure: a call whose heredoc delimiter is never closed
  is one unmatched segment, so its `git status` ran sandboxed and listed the
  masks (2026-08-28).
  A heredoc body can never trigger a match either (only the command word is
  compared), and cannot smuggle a write past the read-only validator (the
  capability comes from the command word, not the channel).
- **Comparison is on source text**, per segment. Only allowlisted harmless
  leading assignments (`LANG`, `TZ`, `NODE_ENV`, `CI`, `NO_COLOR`, …) are
  stripped; any other `VAR=value` prefix keeps the call sandboxed, as do a
  redirection to a file (an fd duplication like `2>&1` is fine), a `cd`,
  `pushd` or `popd` anywhere in the call, and a leading `sudo`, `eval` or
  `xargs`. Wrapper commands are unwrapped — `timeout 5 git status` matches a
  `git status` entry, as do `time`, `nice`, `stdbuf`, `nohup`, `command`,
  `builtin`, `noglob`. `LD_*`, `DYLD_*` and `PATH=` assignments are
  deliberately NOT stripped. Double spaces, quoted arguments, `/bin/ls`, an
  extra argument, and pre-expansion `$VAR` all defeat the match.
- **Every segment must match** for the call to leave the sandbox (CC ≥ 2.1.277;
  earlier, one matching segment freed the whole call, neighbours included —
  fixed as a bug). So `ls -la <dir> | head` and `git log | wc -l` stay
  sandboxed while `ls a; ls b` does not: a pipeline is excluded only when each
  stage has an entry. Nothing in the tool result or the transcript records
  whether a call ran unsandboxed, and a `PreToolUse` hook cannot see the
  decision either: `tool_input.dangerouslyDisableSandbox` is only ever the
  model's own flag, and the exclusion is computed inside the Bash tool's
  `call()`, after the hook (2.1.289 bundle).
- **Shapes observed on 2.1.289 with `ls *` excluded.** Unsandboxed: `ls a &&
  ls b`, `ls a | ls b`, `LANG=C ls …`, a quoted argument, a `\.` escape, a
  double space plus `# comment`, bare `git status --porcelain`. Sandboxed
  despite every segment matching: a glob argument (`ls x/.bash*`), `2>&1`
  (the docs call fd duplication fine), an unquoted `$HOME`.
- **It does not grant approval — but that rarely costs a classifier call.** The
  sandbox auto-allow shortcut is skipped precisely when the command is not
  sandboxed, so deny/ask rules and then the rest of the permission pipeline
  still evaluate it — a `Bash(sudo:*)` deny rule refused a call carrying an
  excluded segment. What catches it next is a **sandbox-independent read-only
  auto-allow** (decision reason `Read-only command is allowed`), which sits
  ahead of the auto-mode classifier and never consults sandbox state: `git
  log`, `git diff`, `git show`, `ls -a` and `find` are allowed locally whether
  sandboxed or not, so unsandboxing them is free. The classifier is reached only
  where that validator refuses. It requires **every** segment to pass, and
  rejects: any subshell or `compound_statement` anywhere, any `&`, any unquoted
  `$VAR`, a `cd` in the same call as any git segment, the git global options
  `-C`/`-c`/`--git-dir`/`--work-tree` (and `--exec-path`, `--config-env`,
  `--bare`, `--attr-source`, `--shallow-file`), and any git segment when
  sandboxing is enabled globally and cwd is not the session's original cwd. So
  the cheap shape is a bare `git <subcommand>` from the original cwd;
  `cd <dir> && git …`, `(cd <dir> && git …)` and `git -C <dir> …` each forfeit
  it. Gating is otherwise equivalent to an agent-declared
  `dangerouslyDisableSandbox`, minus the round-trip.
- **`` !`cmd` `` expansions in a slash command or skill body are matched too**
  (previous section), and `PreToolUse` hooks do not run on that path.
- **Settings reload live.** `excludedCommands`, `autoAllowBashIfSandboxed` and
  `permissions.deny` each took effect mid-session in both directions, with no
  restart — a probe does not need a fresh session.

**Why `git:*` rather than a hook.** The `unsandbox-git-status` plugin — a
`PreToolUse` Bash hook that detected `git status` and flipped
`dangerouslyDisableSandbox` in place — is retired, removed from the `ddaanet`
marketplace and its repo archived. It paired `updatedInput` with
`permissionDecision: "allow"`, pre-approving anything chained alongside the
`git status` (confirmed live: `true && git status --porcelain` ran); the
mechanism, and the rule that a hook which must rewrite returns `updatedInput`
alone, are [[hook-output-channels]]. The exclusion covers every git shape
rather than a best-effort `git status` subset, is honoured in strict mode, and
does not settle the gate.
The one shape the hook caught and the exclusion misses is a git command nested
in `( … )`, `$( … )` or `sh -c`; `cwd-safety` used to manufacture that shape by
wrapping `cd <dir> && …` in a subshell and instead appends `cd <root>` as a
trailing line, so what remains is the genuine residue (`echo "[$(git -C
memory/ddaanet ls-files X)]"`) and git run inside recipes. Do NOT read a clean
unsandboxed `git status` as evidence the sandbox itself is clean — nothing
announces the exclusion in the tool result.

## `ps` cannot see harness background tasks

The agent's Bash runs in its own PID namespace, so `ps` lists only that shell.
A Bash call made with `run_in_background` — and everything it spawns — is
absent from any process listing the agent can take: `ps -eo pid,comm` showed
two `bash` entries while a background `bats` suite was mid-run and went on to
complete normally. The gap is total — `ps aux` returned 5 lines sandboxed
against 141 for the identical command unsandboxed (2026-08-15).

**An empty `ps` is therefore not evidence that a background job died.** The
move it invites is starting the same work in the foreground to check, which
puts two heavy runs on a box that will not take parallel work; the second times
out, and both then look like the first one failing.

**How to apply:** treat the task-completion notification and the task's output
file as the only signals about a background job, and a process listing as no
signal at all. When waiting on one, key the wait on the command's exit rather
than on a line of its output that reads as terminal — a bats summary line
prints before the recipe has run its remaining steps.

## `--resume` dies because the transcript never persisted

Inside the sandbox the API is reachable, but `~/.claude/projects/` (where
session transcripts are written) is read-only. So a `claude --print` turn
completes and returns a `session_id`, yet its transcript is never persisted —
and a following `claude --print --resume <id>` fails with "No conversation found
with session ID". The Agent SDK's `query()` has the same failure (it also spawns
a subprocess and resumes by id). Verified 2026-07-17: `touch
~/.claude/projects/.probe` returns "Read-only file system"; the identical
two-turn resume that failed sandboxed returned `subtype: success` unsandboxed.

**Consequence for eval/harness design:** any two-turn (or N-turn) flow relying
on `--resume` must run **unsandboxed** (`dangerouslyDisableSandbox`, the
`claude:*` exclusion for a top-level call, or my human partner's own shell). A
single-turn connectivity **probe cannot detect this** — it never resumes, so it
passes sandboxed while the real flow cannot. gitlore's
`tests/evals/run-evals.sh` documents this in its header and its probe comment.
Related: [[classifier-denied-self-config]] (the gitdir is likewise
write-blocked, which is why the memory-message IPC file lives in the parent
worktree).

**So `just prerelease` and `just release` must run unsandboxed** — `release`
depends on `prerelease`, which runs `just evals`, which is exactly such a flow,
and `claude:*` does not reach a `claude` inside a recipe. Sandboxed it fails at
`pass^5` in a way that looks like it indicts the scenarios or the code, and does
not: every run of an affected scenario fails identically, 0/5 rather than
flaky. Two signatures name the cause. Every two-turn scenario reports `No
conversation found with session ID: … turn 2 yielded no result`, and
`05-recall` reports `no session transcript captured` from its assertion, which
reads the transcript to tell an injected body from a self-issued Read. A third
tell is upstream of the evals and visible on every trial: `direnv: error open …
read-only file system` from the install step. Diagnose from these before
suspecting a scenario — and note that piping the gate into `tail` reports the
pipeline's exit status, so a failing run looks like exit 0.

## zellij IPC socket is outside the sandbox

In a zellij session (`ZELLIJ_SESSION_NAME=...`), calling `zellij run
--floating`, `zellij action new-pane`, etc. from the Bash tool returns "There
is no active session!" under the default sandbox, even with explicit `--session
<name>`. The session exists (`zellij list-sessions` shows "current") but
zellij's IPC unix socket (typically under `/tmp/zellij-<uid>/zellij-<session>/`)
is not reachable from the sandbox.

**Fix:** call Bash with `dangerouslyDisableSandbox: true`. The first symptom is
"no active session" from any zellij subcommand that needs to talk to the
running daemon (verified with `zellij --version` 0.44.3).

**Affects:** the `/revdiff:revdiff` skill launcher (`launch-revdiff.sh`) — it
picks the zellij branch when `ZELLIJ` is set and falls silently because the
launcher pipes `zellij run` stderr to `/dev/null`, so sandbox failure looks like
"exit 1 with no output"; and any future skill/script driving zellij from inside
CC. Once sandbox-disabled, the floating pane appears immediately.

## Sandboxed `claude -p` silently drops every `SessionStart` hook

A `claude -p <prompt>` spawned from inside the command sandbox needs to `mkdir`
a per-hook scratch dir under `~/.claude/session-env/<session-id>/` before
running each `SessionStart` hook, and that path sits outside the sandbox's
write allowlist. Every registered `SessionStart` hook across every plugin —
not just the one you're chasing — then fails with `EROFS: read-only file
system, mkdir '.../session-env/<id>'`. This surfaces only in
`[ERROR]`/`[DEBUG]` log lines (`-d hooks --debug-file <path>`); the turn itself
completes and returns its normal reply, so nothing in the visible output
signals that every `SessionStart` side effect was skipped — the strictest
silent failure in this file, and the reason `claude:*` is in the baseline
exclusions.

Symptom: a fix that depends on a `SessionStart` hook's side effect appears to
do nothing even after the documented remedy. Confirmed 2026-08-11: gitlore's
`git config gitlore.hooksDir` (repointed to the current plugin version every
`SessionStart`, [[stale-plugin-code]]) stayed pinned at the old version through
two sandboxed `claude -p ping` calls in the target repo — the standard "start a
session there" remedy — and only updated once retried with
`dangerouslyDisableSandbox: true`.

**How to apply:** a top-level `claude -p` is unsandboxed by the exclusion; one
inside a recipe (`just evals`, `just release`) or a script is not, and needs the
whole call unsandboxed whenever its *point* is a hook's side effect. A
sandboxed one is fine only when you need just the model's textual reply.
[[posttooluse-print-mode]] establishes that `SessionStart` fires under
`--print` at all — that finding was made unsandboxed; this is the
sandbox-specific caveat on top of it.

## Sandbox setup itself fails: `apply-seccomp: unshare(CLONE_NEWUSER): Invalid argument`

Distinct from every symptom above (a specific path/socket denied): the
sandboxing mechanism can fail to initialize at all, so an arbitrary command —
including a trivial read-only `grep` on a local file — errors immediately with
`apply-seccomp: unshare(CLONE_NEWUSER): Invalid argument` and no other output,
before the command itself runs. None of the file/lock/socket diagnostics above
apply, since the error names no path or operation.

**It is transient, so the remedy is a plain retry, not an unsandboxed one.**
The standing-policy cause that would make retrying pointless does not obtain on
the ddaanet box (`kernel.unprivileged_userns_clone = 1`,
`user.max_user_namespaces = 2147483647`); `unshare(2)` returns `EINVAL` for
`CLONE_NEWUSER` in a multithreaded caller, a per-invocation condition, and the
box OOMs near 2 GB, so a transient failure at namespace setup is the live
reading. Confirmed 2026-08-15 three times: the byte-identical command retried
unchanged succeeded. The retry is not perfectly reliable — one multi-statement
command failed twice consecutively and went through once split into smaller
single-purpose calls.

**How to apply:** retry the identical command once, unchanged. If it fails the
same way immediately, split it. Escape the sandbox only after both, rather than
giving it up on first sight.

## `/tmp` reads full while `du` finds little

A write under `/tmp` (where `$TMPDIR` lives) fails with `No space left on
device` — git spells it `Out of diskspace` — while `df` shows the tmpfs at
100% and `du` over every visible file, root included, accounts for a fraction
of it, with `lsof +L1` finding no deleted-but-open files. Observed on the
ddaanet box 2026-09-01: 2.0G tmpfs, 100% used, ~374M visible. The unreachable
remainder sits with other live Claude sessions: each sandbox holds bind-mounts
of subtrees of the same tmpfs superblock in its own mount namespace, so pages
referenced only there are invisible to any `du` run in this one, and they free
only when those sessions exit.

**How to apply:** reclaim the visible part and skip the forensics —
`/tmp/claude-1000` accumulates `uv-cache`, `npm-cache`, `dprint-cache` and
hundreds of leaked `gitlore-test.*`/`bats-run-*` scratch dirs from killed
suites (a killed bats run skips teardown); clearing those is safe, and is the
recoverable share whatever holds the rest. Chasing the gap with
`lsof`/`findmnt` costs calls and changes nothing.
